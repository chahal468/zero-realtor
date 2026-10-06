# Python 3 & SQLAlchemy Endpoint Development

## Quick Start

### Installation
```bash
pip install fastapi uvicorn sqlalchemy psycopg2-binary alembic pydantic
```

### Basic Project Structure
```
my_api/
├── app/
│   ├── __init__.py
│   ├── main.py
│   ├── database.py
│   ├── models.py
│   ├── schemas.py
│   ├── crud.py
│   └── endpoints.py
├── alembic/
├── requirements.txt
└── .env
```

## 1. Database Setup

### Database Configuration
```python
# database.py
from sqlalchemy import create_engine
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import sessionmaker
import os

DATABASE_URL = os.getenv("DATABASE_URL", "postgresql://user:pass@localhost/mydb")

engine = create_engine(DATABASE_URL)
SessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=engine)
Base = declarative_base()

def get_db():
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()
```

### Model Definition
```python
# models.py
from sqlalchemy import Column, Integer, String, Boolean, DateTime, ForeignKey, Text
from sqlalchemy.orm import relationship
from datetime import datetime
from .database import Base

class User(Base):
    __tablename__ = "users"
    
    id = Column(Integer, primary_key=True, index=True)
    name = Column(String(100), nullable=False)
    email = Column(String(100), unique=True, index=True, nullable=False)
    password_hash = Column(String(255), nullable=False)
    is_active = Column(Boolean, default=True)
    created_at = Column(DateTime, default=datetime.utcnow)
    
    # Relationships
    posts = relationship("Post", back_populates="author", cascade="all, delete-orphan")

class Post(Base):
    __tablename__ = "posts"
    
    id = Column(Integer, primary_key=True, index=True)
    title = Column(String(200), nullable=False)
    content = Column(Text, nullable=False)
    author_id = Column(Integer, ForeignKey("users.id"), nullable=False)
    is_published = Column(Boolean, default=False)
    created_at = Column(DateTime, default=datetime.utcnow)
    
    # Relationships
    author = relationship("User", back_populates="posts")
```

### Pydantic Schemas
```python
# schemas.py
from pydantic import BaseModel, EmailStr
from typing import List, Optional
from datetime import datetime

# User schemas
class UserBase(BaseModel):
    name: str
    email: EmailStr

class UserCreate(UserBase):
    password: str

class UserUpdate(BaseModel):
    name: Optional[str] = None
    email: Optional[EmailStr] = None
    is_active: Optional[bool] = None

class User(UserBase):
    id: int
    is_active: bool
    created_at: datetime
    
    class Config:
        from_attributes = True

# Post schemas
class PostBase(BaseModel):
    title: str
    content: str

class PostCreate(PostBase):
    pass

class PostUpdate(BaseModel):
    title: Optional[str] = None
    content: Optional[str] = None
    is_published: Optional[bool] = None

class Post(PostBase):
    id: int
    author_id: int
    is_published: bool
    created_at: datetime
    author: User
    
    class Config:
        from_attributes = True
```

## 2. CRUD Operations

### Base CRUD Class
```python
# crud.py
from typing import Any, Dict, Generic, List, Optional, Type, TypeVar, Union
from fastapi.encoders import jsonable_encoder
from pydantic import BaseModel
from sqlalchemy.orm import Session

ModelType = TypeVar("ModelType", bound=Base)
CreateSchemaType = TypeVar("CreateSchemaType", bound=BaseModel)
UpdateSchemaType = TypeVar("UpdateSchemaType", bound=BaseModel)

class CRUDBase(Generic[ModelType, CreateSchemaType, UpdateSchemaType]):
    def __init__(self, model: Type[ModelType]):
        self.model = model

    def get(self, db: Session, id: Any) -> Optional[ModelType]:
        return db.query(self.model).filter(self.model.id == id).first()

    def get_multi(self, db: Session, *, skip: int = 0, limit: int = 100) -> List[ModelType]:
        return db.query(self.model).offset(skip).limit(limit).all()

    def create(self, db: Session, *, obj_in: CreateSchemaType) -> ModelType:
        obj_in_data = jsonable_encoder(obj_in)
        db_obj = self.model(**obj_in_data)
        db.add(db_obj)
        db.commit()
        db.refresh(db_obj)
        return db_obj

    def update(
        self,
        db: Session,
        *,
        db_obj: ModelType,
        obj_in: Union[UpdateSchemaType, Dict[str, Any]]
    ) -> ModelType:
        obj_data = jsonable_encoder(db_obj)
        if isinstance(obj_in, dict):
            update_data = obj_in
        else:
            update_data = obj_in.dict(exclude_unset=True)
        
        for field in obj_data:
            if field in update_data:
                setattr(db_obj, field, update_data[field])
        
        db.add(db_obj)
        db.commit()
        db.refresh(db_obj)
        return db_obj

    def remove(self, db: Session, *, id: int) -> ModelType:
        obj = db.query(self.model).get(id)
        db.delete(obj)
        db.commit()
        return obj
```

### User CRUD
```python
# crud.py (continued)
from .models import User, Post
from .schemas import UserCreate, UserUpdate, PostCreate, PostUpdate
from .security import get_password_hash

class CRUDUser(CRUDBase[User, UserCreate, UserUpdate]):
    def get_by_email(self, db: Session, *, email: str) -> Optional[User]:
        return db.query(User).filter(User.email == email).first()

    def create(self, db: Session, *, obj_in: UserCreate) -> User:
        db_obj = User(
            email=obj_in.email,
            name=obj_in.name,
            password_hash=get_password_hash(obj_in.password),
        )
        db.add(db_obj)
        db.commit()
        db.refresh(db_obj)
        return db_obj

class CRUDPost(CRUDBase[Post, PostCreate, PostUpdate]):
    def get_with_author(self, db: Session, *, id: int) -> Optional[Post]:
        return db.query(Post).filter(Post.id == id).first()

    def get_multi_by_author(
        self, db: Session, *, author_id: int, skip: int = 0, limit: int = 100
    ) -> List[Post]:
        return (
            db.query(self.model)
            .filter(Post.author_id == author_id)
            .offset(skip)
            .limit(limit)
            .all()
        )

    def create_with_author(
        self, db: Session, *, obj_in: PostCreate, author_id: int
    ) -> Post:
        obj_in_data = jsonable_encoder(obj_in)
        db_obj = self.model(**obj_in_data, author_id=author_id)
        db.add(db_obj)
        db.commit()
        db.refresh(db_obj)
        return db_obj

# Create instances
user = CRUDUser(User)
post = CRUDPost(Post)
```

## 3. FastAPI Endpoints

### Main Application
```python
# main.py
from fastapi import FastAPI, Depends, HTTPException, status
from fastapi.middleware.cors import CORSMiddleware
from sqlalchemy.orm import Session
from .database import get_db
from .models import User, Post
from .schemas import User as UserSchema, Post as PostSchema
from .schemas import UserCreate, UserUpdate, PostCreate, PostUpdate
from .crud import user as user_crud, post as post_crud

app = FastAPI(title="Python SQLAlchemy API")

# CORS
app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"],
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)

@app.get("/")
def read_root():
    return {"message": "Welcome to SQLAlchemy API"}

# User endpoints
@app.post("/users/", response_model=UserSchema, status_code=status.HTTP_201_CREATED)
def create_user(user: UserCreate, db: Session = Depends(get_db)):
    db_user = user_crud.get_by_email(db, email=user.email)
    if db_user:
        raise HTTPException(status_code=400, detail="Email already registered")
    return user_crud.create(db=db, obj_in=user)

@app.get("/users/", response_model=list[UserSchema])
def read_users(skip: int = 0, limit: int = 100, db: Session = Depends(get_db)):
    users = user_crud.get_multi(db, skip=skip, limit=limit)
    return users

@app.get("/users/{user_id}", response_model=UserSchema)
def read_user(user_id: int, db: Session = Depends(get_db)):
    db_user = user_crud.get(db, id=user_id)
    if db_user is None:
        raise HTTPException(status_code=404, detail="User not found")
    return db_user

@app.put("/users/{user_id}", response_model=UserSchema)
def update_user(
    user_id: int, user: UserUpdate, db: Session = Depends(get_db)
):
    db_user = user_crud.get(db, id=user_id)
    if db_user is None:
        raise HTTPException(status_code=404, detail="User not found")
    return user_crud.update(db=db, db_obj=db_user, obj_in=user)

@app.delete("/users/{user_id}", status_code=status.HTTP_204_NO_CONTENT)
def delete_user(user_id: int, db: Session = Depends(get_db)):
    db_user = user_crud.get(db, id=user_id)
    if db_user is None:
        raise HTTPException(status_code=404, detail="User not found")
    user_crud.remove(db=db, id=user_id)

# Post endpoints
@app.post("/posts/", response_model=PostSchema, status_code=status.HTTP_201_CREATED)
def create_post(post: PostCreate, author_id: int, db: Session = Depends(get_db)):
    # Verify user exists
    db_user = user_crud.get(db, id=author_id)
    if db_user is None:
        raise HTTPException(status_code=404, detail="Author not found")
    
    return post_crud.create_with_author(db=db, obj_in=post, author_id=author_id)

@app.get("/posts/", response_model=list[PostSchema])
def read_posts(skip: int = 0, limit: int = 100, db: Session = Depends(get_db)):
    posts = post_crud.get_multi(db, skip=skip, limit=limit)
    return posts

@app.get("/posts/{post_id}", response_model=PostSchema)
def read_post(post_id: int, db: Session = Depends(get_db)):
    db_post = post_crud.get_with_author(db, id=post_id)
    if db_post is None:
        raise HTTPException(status_code=404, detail="Post not found")
    return db_post

@app.get("/users/{user_id}/posts", response_model=list[PostSchema])
def read_user_posts(user_id: int, skip: int = 0, limit: int = 100, db: Session = Depends(get_db)):
    # Verify user exists
    db_user = user_crud.get(db, id=user_id)
    if db_user is None:
        raise HTTPException(status_code=404, detail="User not found")
    
    posts = post_crud.get_multi_by_author(db, author_id=user_id, skip=skip, limit=limit)
    return posts

@app.put("/posts/{post_id}", response_model=PostSchema)
def update_post(
    post_id: int, post: PostUpdate, db: Session = Depends(get_db)
):
    db_post = post_crud.get(db, id=post_id)
    if db_post is None:
        raise HTTPException(status_code=404, detail="Post not found")
    return post_crud.update(db=db, db_obj=db_post, obj_in=post)

@app.delete("/posts/{post_id}", status_code=status.HTTP_204_NO_CONTENT)
def delete_post(post_id: int, db: Session = Depends(get_db)):
    db_post = post_crud.get(db, id=post_id)
    if db_post is None:
        raise HTTPException(status_code=404, detail="Post not found")
    post_crud.remove(db=db, id=post_id)
```

## 4. Advanced Query Examples

### Complex Queries
```python
# crud_advanced.py
from sqlalchemy.orm import Session, joinedload
from sqlalchemy import and_, or_, func, desc
from .models import User, Post

class AdvancedQueries:
    @staticmethod
    def get_users_with_post_count(db: Session, limit: int = 10):
        """Get users with their post count"""
        return (
            db.query(User, func.count(Post.id).label('post_count'))
            .outerjoin(Post)
            .group_by(User.id)
            .order_by(func.count(Post.id).desc())
            .limit(limit)
            .all()
        )

    @staticmethod
    def search_posts(db: Session, query: str, author_id: int = None):
        """Search posts by title/content"""
        filters = [
            or_(
                Post.title.ilike(f"%{query}%"),
                Post.content.ilike(f"%{query}%")
            )
        ]
        
        if author_id:
            filters.append(Post.author_id == author_id)
        
        return (
            db.query(Post)
            .filter(and_(*filters))
            .order_by(desc(Post.created_at))
            .all()
        )

    @staticmethod
    def get_posts_with_author(db: Session, published_only: bool = True):
        """Get posts with author data loaded"""
        query = db.query(Post).options(joinedload(Post.author))
        
        if published_only:
            query = query.filter(Post.is_published == True)
        
        return query.all()

    @staticmethod
    def get_user_stats(db: Session, user_id: int):
        """Get user statistics"""
        return (
            db.query(
                func.count(Post.id).label('total_posts'),
                func.sum(func.case([(Post.is_published == True, 1)], else_=0)).label('published_posts')
            )
            .filter(Post.author_id == user_id)
            .first()
        )
```

### Advanced Endpoints
```python
# endpoints_advanced.py
from fastapi import APIRouter, Depends, HTTPException, Query
from sqlalchemy.orm import Session
from typing import Optional
from .database import get_db
from .crud_advanced import AdvancedQueries
from .schemas import Post, User

router = APIRouter()

@router.get("/users/stats/{user_id}")
def get_user_statistics(user_id: int, db: Session = Depends(get_db)):
    """Get user statistics"""
    stats = AdvancedQueries.get_user_stats(db, user_id)
    if not stats or stats.total_posts == 0:
        raise HTTPException(status_code=404, detail="User or posts not found")
    
    return {
        "user_id": user_id,
        "total_posts": stats.total_posts,
        "published_posts": stats.published_posts or 0,
        "draft_posts": stats.total_posts - (stats.published_posts or 0)
    }

@router.get("/users/with-post-count")
def get_users_with_post_count(limit: int = Query(10, le=100), db: Session = Depends(get_db)):
    """Get users with their post counts"""
    results = AdvancedQueries.get_users_with_post_count(db, limit)
    
    return [
        {
            "user": user.__dict__,
            "post_count": post_count
        }
        for user, post_count in results
    ]

@router.get("/posts/search")
def search_posts(
    q: str = Query(..., min_length=2),
    author_id: Optional[int] = Query(None),
    db: Session = Depends(get_db)
):
    """Search posts"""
    posts = AdvancedQueries.search_posts(db, q, author_id)
    return posts

@router.get("/posts/with-author")
def get_posts_with_author(
    published_only: bool = Query(True),
    db: Session = Depends(get_db)
):
    """Get posts with author information"""
    return AdvancedQueries.get_posts_with_author(db, published_only)
```

## 5. Database Migrations

### Initialize Alembic
```bash
alembic init alembic
```

### Configure Alembic
```python
# alembic/env.py
from alembic import context
from sqlalchemy import engine_from_config, pool
from logging.config import fileConfig
from app.models import Base  # Import your models
from app.database import DATABASE_URL

# Add your model's MetaData object here for 'autogenerate' support
target_metadata = Base.metadata

def run_migrations_online():
    configuration = context.config
    configuration.set_main_option("sqlalchemy.url", DATABASE_URL)
    
    connectable = engine_from_config(
        configuration.get_section(configuration.config_ini_section),
        prefix="sqlalchemy.",
        poolclass=pool.NullPool,
    )

    with connectable.connect() as connection:
        context.configure(
            connection=connection, target_metadata=target_metadata
        )

        with context.begin_transaction():
            context.run_migrations()
```

### Create and Run Migrations
```bash
# Create migration
alembic revision --autogenerate -m "Create users and posts tables"

# Apply migrations
alembic upgrade head

# Downgrade
alembic downgrade -1
```

## 6. Error Handling & Validation

### Custom Exceptions
```python
# exceptions.py
from fastapi import HTTPException, status

class UserNotFoundError(HTTPException):
    def __init__(self, user_id: int):
        super().__init__(
            status_code=status.HTTP_404_NOT_FOUND,
            detail=f"User with id {user_id} not found"
        )

class EmailAlreadyExistsError(HTTPException):
    def __init__(self, email: str):
        super().__init__(
            status_code=status.HTTP_400_BAD_REQUEST,
            detail=f"Email {email} already registered"
        )

class PostNotFoundError(HTTPException):
    def __init__(self, post_id: int):
        super().__init__(
            status_code=status.HTTP_404_NOT_FOUND,
            detail=f"Post with id {post_id} not found"
        )
```

### Enhanced Endpoints with Error Handling
```python
# endpoints_safe.py
from fastapi import APIRouter, Depends, HTTPException, status
from sqlalchemy.orm import Session
from .database import get_db
from .schemas import UserCreate, UserUpdate
from .crud import user as user_crud
from .exceptions import UserNotFoundError, EmailAlreadyExistsError

router = APIRouter()

@router.post("/users/", response_model=User, status_code=status.HTTP_201_CREATED)
def create_user_safe(user: UserCreate, db: Session = Depends(get_db)):
    """Create user with proper error handling"""
    try:
        # Check if email already exists
        existing_user = user_crud.get_by_email(db, email=user.email)
        if existing_user:
            raise EmailAlreadyExistsError(user.email)
        
        return user_crud.create(db=db, obj_in=user)
    
    except HTTPException:
        raise
    except Exception as e:
        raise HTTPException(
            status_code=status.HTTP_500_INTERNAL_SERVER_ERROR,
            detail=f"Internal server error: {str(e)}"
        )

@router.get("/users/{user_id}", response_model=User)
def read_user_safe(user_id: int, db: Session = Depends(get_db)):
    """Get user with proper error handling"""
    user = user_crud.get(db, id=user_id)
    if not user:
        raise UserNotFoundError(user_id)
    return user
```

## 7. Testing

### Test Setup
```python
# conftest.py
import pytest
from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker
from fastapi.testclient import TestClient
from app.main import app
from app.database import get_db, Base

# Test database
SQLALCHEMY_DATABASE_URL = "sqlite:///./test.db"
engine = create_engine(SQLALCHEMY_DATABASE_URL, connect_args={"check_same_thread": False})
TestingSessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=engine)

@pytest.fixture(scope="function")
def db_session():
    Base.metadata.create_all(bind=engine)
    session = TestingSessionLocal()
    try:
        yield session
    finally:
        session.close()
        Base.metadata.drop_all(bind=engine)

@pytest.fixture(scope="function")
def client(db_session):
    def override_get_db():
        try:
            yield db_session
        finally:
            pass
    
    app.dependency_overrides[get_db] = override_get_db
    with TestClient(app) as test_client:
        yield test_client
    app.dependency_overrides.clear()
```

### Test Examples
```python
# test_endpoints.py
def test_create_user(client):
    response = client.post(
        "/users/",
        json={
            "name": "Test User",
            "email": "test@example.com",
            "password": "password123"
        }
    )
    assert response.status_code == 201
    data = response.json()
    assert data["name"] == "Test User"
    assert data["email"] == "test@example.com"
    assert "id" in data

def test_get_users(client):
    # Create a user first
    client.post(
        "/users/",
        json={
            "name": "Test User",
            "email": "test@example.com",
            "password": "password123"
        }
    )
    
    response = client.get("/users/")
    assert response.status_code == 200
    data = response.json()
    assert len(data) > 0

def test_get_user_not_found(client):
    response = client.get("/users/999")
    assert response.status_code == 404
    assert "not found" in response.json()["detail"].lower()
```

## 8. Running the Application

### Development Server
```bash
uvicorn app.main:app --reload --host 0.0.0.0 --port 8000
```

### Production Server
```bash
uvicorn app.main:app --host 0.0.0.0 --port 8000 --workers 4
```

### Environment Variables (.env)
```bash
DATABASE_URL=postgresql://user:password@localhost/mydb
SECRET_KEY=your-secret-key-here
DEBUG=True
```

## 9. API Documentation

Once running, visit:
- **Swagger UI**: `http://localhost:8000/docs`
- **ReDoc**: `http://localhost:8000/redoc`

## 10. Common Patterns

### Pagination
```python
from pydantic import BaseModel
from typing import List, Generic, TypeVar

T = TypeVar("T")

class PaginationParams(BaseModel):
    page: int = 1
    size: int = 20
    
    @property
    def skip(self) -> int:
        return (self.page - 1) * self.size

class PaginatedResponse(BaseModel, Generic[T]):
    items: List[T]
    total: int
    page: int
    size: int
    pages: int

@app.get("/users/paginated", response_model=PaginatedResponse[User])
def get_users_paginated(
    pagination: PaginationParams = Depends(),
    db: Session = Depends(get_db)
):
    total = db.query(User).count()
    users = user_crud.get_multi(db, skip=pagination.skip, limit=pagination.size)
    
    pages = (total + pagination.size - 1) // pagination.size
    
    return PaginatedResponse(
        items=users,
        total=total,
        page=pagination.page,
        size=pagination.size,
        pages=pages
    )
```

### Filtering
```python
from sqlalchemy import and_, or_

@app.get("/posts/filter")
def filter_posts(
    author_id: Optional[int] = None,
    published: Optional[bool] = None,
    search: Optional[str] = None,
    db: Session = Depends(get_db)
):
    query = db.query(Post)
    
    if author_id:
        query = query.filter(Post.author_id == author_id)
    
    if published is not None:
        query = query.filter(Post.is_published == published)
    
    if search:
        query = query.filter(
            or_(
                Post.title.ilike(f"%{search}%"),
                Post.content.ilike(f"%{search}%")
            )
        )
    
    return query.all()
```

This guide provides everything you need to build robust RESTful APIs with Python 3 and SQLAlchemy, from basic CRUD to advanced patterns like pagination, filtering, and comprehensive error handling.
