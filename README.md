# Remotebricspython_assignment
from fastapi import FastAPI, Body, HTTPException
from fastapi.security import OAuth2PasswordBearer
from pymongo import MongoClient
from passlib.hash import bcrypt

# Replace with your actual MongoDB connection details
client = MongoClient("mongodb://localhost:27017/")
db = client["user_management"]  # Replace "user_management" with your database name
users_collection = db["users"]
linked_ids_collection = db["linked_ids"]

# Configure security for login API
oauth2_scheme = OAuth2PasswordBearer(tokenUrl="/login")

app = FastAPI()


# Helper function to hash passwords securely
def hash_password(password: str):
    return bcrypt.hash(password)


# Registration API
@app.post("/register")
async def register(username: str = Body(...), email: str = Body(...), password: str = Body(...)):
    """
    Registers a new user in the system.
    """
    existing_user = users_collection.find_one({"email": email})
    if existing_user:
        raise HTTPException(status_code=400, detail="Email already exists")

    hashed_password = hash_password(password)
    user_data = {"username": username, "email": email, "password": hashed_password}
    users_collection.insert_one(user_data)
    return {"message": "User registered successfully"}


# Login API (basic implementation, consider using JWT for better security)
@app.post("/login")
async def login(email: str = Body(...), password: str = Body(...)):
    """
    Authenticates a user and returns a simple success message.
    """
    user = users_collection.find_one({"email": email})
    if not user:
        raise HTTPException(status_code=401, detail="Invalid credentials")

    if not bcrypt.verify(password, user["password"]):
        raise HTTPException(status_code=401, detail="Invalid credentials")

    return {"message": "Login successful"}


# Linking ID API
@app.post("/link-id", security=[oauth2_scheme])
async def link_id(id_value: str = Body(...), token: str = Depends(oauth2_scheme)):
    """
    Allows a logged-in user to link an ID to their account.
    """
    # Get user details from token (implementation omitted for brevity)
    user_email = get_user_email_from_token(token)  # Replace with your token decoding logic

    existing_link = linked_ids_collection.find_one({"user_email": user_email})
    if existing_link:
        raise HTTPException(status_code=400, detail="ID already linked")

    linked_ids_collection.insert_one({"user_email": user_email, "id_value": id_value})
    return {"message": "ID linked successfully"}


# Placeholder functions for Joins and Chain Delete (further development required)
@app.get("/joins")
async def joins():
    """
    Placeholder function for joining data from multiple collections.
    This requires defining specific logic based on your data structure and query needs.
    """
    pass


@app.delete("/user/{user_id}")
async def delete_user(user_id: str):
    """
    Placeholder function for chain delete functionality.
    This requires implementing logic to cascade deletion across collections
    based on user ID or other linking fields.
    """
    pass

if __name__ == "__main__":
    import uvicorn
    uvicorn.run(app, host="0.0.0.0", port=8000)
