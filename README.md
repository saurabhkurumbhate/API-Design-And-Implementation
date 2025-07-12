# API-Design-And-Implementation
!pip install fastapi uvicorn
from fastapi import FastAPI, HTTPException
from pydantic import BaseModel
from typing import List, Dict

app = FastAPI()

# Models
class Book(BaseModel):
    id: int
    title: str
    author: str

class Review(BaseModel):
    reviewer: str
    text: str
    rating: int  # 1 to 5

# In-memory data storage
books_db: List[Book] = []
reviews_db: Dict[int, List[Review]] = {}

# API Routes

@app.get("/books", response_model=List[Book])
def list_books():
    return books_db

@app.post("/books", response_model=Book)
def add_book(book: Book):
    for b in books_db:
        if b.id == book.id:
            raise HTTPException(status_code=400, detail="Book with this ID already exists.")
    books_db.append(book)
    return book

@app.get("/books/{id}/reviews", response_model=List[Review])
def get_reviews(id: int):
    if id not in reviews_db:
        return []  # No reviews yet
    return reviews_db[id]

@app.post("/books/{id}/reviews", response_model=Review)
def add_review(id: int, review: Review):
    if not any(b.id == id for b in books_db):
        raise HTTPException(status_code=404, detail="Book not found.")
    if id not in reviews_db:
        reviews_db[id] = []
    reviews_db[id].append(review)


    return review
