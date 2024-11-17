<h1 id="library-management-system">Library Management System with Token Authentication</h1>

<!-- TABLE OF CONTENTS -->
<details>
  <summary>Table of Contents</summary>
  <ol>
    <li>
      <a href="#library-management-system">About The System/Project</a>
    </li>
    <li>
      <a href="#getting-started">Getting Started</a>
      <ul>
        <li><a href="#prerequisites">Prerequisites</a></li>
        <li><a href="#installing">Installing</a></li>
      </ul>
    </li>
    <li><a href="#implementation">Implementation</a>
        <ul>
        <li><a href="#user-endpoints">User Endpoints</a></li>
        <li><a href="#author-endpoints">Author Endpoints</a></li>
        <li><a href="#book-endpoints">Book Endpoints</a></li>
        <li><a href="#book-author-relationship-endpoints">Book-Author Relationship Endpoints</a></li>
        <li><a href="#search-endpoints">Search Endpoints</a></li>
        <li><a href="#catalog-endpoints">Catalog Endpoints</a></li>
      </ul>
    </li>
    <li><a href="#token-management">Token Management</a></li>
    <li><a href="#troubleshooting--faq">Troubleshooting / FAQ</a></li>
    <li><a href="#project-information">Project Information</a></li>
  </ol>
</details>

## About the Project

The Library Management System offers a secure, efficient solution for managing books, authors, users, and their associations, along with robust search and catalog capabilities. It supports full CRUD (Create, Read, Update, Delete) operations for user management—enabling registration, authentication, viewing, updating, and deletion of user profiles—as well as for books, authors, and book-author associations. Users can easily search for and catalog books and authors, ensuring quick access to information. Token-based authentication with validation and usage tracking ensures that only authorized users can perform operations. The book-author relationship table enhances flexibility by linking books to their respective authors. This system is designed to simplify and secure library data management while maintaining strong security standards. 

<p align="right">(<a href="#library-management-system">back to top</a>)</p>

## Getting Started

### Prerequisites

- XAMPP
- SQLyog (or phpMyAdmin)
- JWT PHP Library
- Node.js
- Composer
- PHP (version 7.2 or higher)
- Slim Framework
- ThunderClient

### Installing

1. **Clone the Repository**

   ```bash
   git clone https://github.com/github_username/library_4a.git
   cd /path/to/xampp/htdocs/library_4a

   ```

2. **Install Dependencies**

   - Use Composer to install PHP dependencies:

   ```bash
   composer install

   ```

3. **Set Up Database**

   - Open SQLyog or phpMyAdmin and create a new database called `library`.
   - Run the following SQL queries to create the required tables:

   ```sql
   CREATE TABLE users (
       userid INT(9) NOT NULL AUTO_INCREMENT,
       username CHAR(255) NOT NULL,
       password TEXT NOT NULL,
       PRIMARY KEY (userid)
   );

   CREATE TABLE authors (
       authorid INT(9) NOT NULL AUTO_INCREMENT,
       name CHAR(255) NOT NULL,
       PRIMARY KEY (authorid)
   );

   CREATE TABLE books (
       bookid INT(9) NOT NULL AUTO_INCREMENT,
       title CHAR(255) NOT NULL,
       PRIMARY KEY (bookid)
   );

   CREATE TABLE books_authors (
       collectionid INT(9) NOT NULL AUTO_INCREMENT,
       bookid INT(9) NOT NULL,
       authorid INT(9) NOT NULL,
       PRIMARY KEY (collectionid)
   );

   CREATE TABLE tokens (
       token VARCHAR(512) PRIMARY KEY,
       used_at DATETIME NOT NULL
   );
   ```

4. **Configure Database Connection**

   - Modify the connection details in the index.php file as specified :

   ```php
   <?php
   $servername = "localhost";
   $username = "root";
   $password = "password";
   $dbname = "library";
   ?>
   ```

   Substitute these values with your actual database settings to establish a connection to the library database.

5. **Start XAMPP Server**

   - Make sure that both Apache and MySQL are active/running in the XAMPP control panel.

6. **Testing the Application**
   - You can now test the CRUD operations and authentication endpoints using API testing tools such as Postman or Thunder Client(default testing tool i used).

<p align="right">(<a href="#library-management-system">back to top</a>)</p>

## Implementation

<h3 id="user-endpoints">1. User Endpoints</h3>

**a. User Registration** - creates a new user account using a hashed password and a unique username.

- **Endpoint:** `/user/register`
- **Method:** `POST`
- **Sample Payload:**

  ```json
  {
    "username": "type your username ",
    "password": "type your password"
  }
  ```

- **Expected Response:**

  - **Success:**

    ```json
    {
      "status": "success",
      "data": null
    }
    ```

  - **Failure:**

    ```json
    {
      "status": "fail",
      "data": {
        "title": "(Error Message Here)"
      }
    }
    ```

**b. User Authentication** - creates a JWT token for session management and authenticates a user.

- **Endpoint:** `/user/authenticate`
- **Method:** `POST`
- **Sample Payload:**

  ```json
  {
    "username": "existing username",
    "password": "existing Password"
  }
  ```

- **Expected Response:**

  - **Success:**

    ```json
    {
      "status": "success",
      "token": "place jwtToken Here",
      "data": null
    }
    ```

  - **Failure:**

    ```json
    {
      "status": "fail",
      "data": {
        "title": "Authentication Failed"
      }
    }
    ```

**c. Display Users** - obtains a list of every user in the system; a valid token is needed.

- **Endpoint:** `/user/display`
- **Method:** `GET`
- **Headers:** `Authorization: Bearer <Enter the jwtToken that was generated by the users here>`

- **Expected Response:**

  - **Success:**

    ```json
    {
      "status": "success",
      "data": [
        {
          "userid": 1,
          "username": "username"
        }
      ]
    }
    ```

  - **Failure:** Token Already Used

    ```json
    {
      "status": "fail",
      "data": {
        "title": "Token has already been used"
      }
    }
    ```

  - **Failure:** Invalid or Expired Token

    ```json
    {
      "status": "fail",
      "data": {
        "title": "Invalid or expired token"
      }
    }
    ```

**d. Update User Information** - updates the user's password and/or username; a working token is needed.

- **Endpoint:** `/user/update`
- **Method:** `PUT`
- **Headers:** `Authorization: Bearer <insert generated jwtTokenHere from the users/authenticate>`
- **Sample Payload:**

  ```json
  {
    "username": "updated Username",
    "password": "new Password"
  }
  ```

- **Expected Response:**

  - **Success:**

    ```json
    {
      "status": "success",
      "data": null
    }
    ```

  - **Failure:** A suitable error notice will appear if the new username is already taken, if there is nothing to update, or if the token is invalid, expired, or already used.
    
**e. Delete User** - removes the verified user's account from the database; a working token is needed.

- **Endpoint:** `/user/delete`
- **Method:** `DELETE`
- **Headers:** `Authorization: Bearer <insert generated jwtTokenHere from the users/authenticate>`

  ```json
  {
    "Token": "place jwtToken Here",
    "userid": "place userid"
  }
  ```

- **Expected Response:**

  - **Success:**

    ```json
    {
      "status": "success",
      "Token": "generated token",
      "data": null
    }
    ```

  - **Failure:** If the user doesn’t exist, or if the token is invalid, expired, or already used, an appropriate error message.

<p align="right">(<a href="#library-management-system">back to top</a>)</p>

<h3 id="author-endpoints">2. Author Endpoints</h3>

**a. Register Author** - register/add a new author to the database.

- **Endpoint:** `/author/register`
- **Method:** `POST`
- **Headers:** `Authorization: Bearer <insert generated jwtTokenHere from the users/authenticate>`
- **Sample Payload:**

  ```json
  {
    "Token":"place jwtToken Here",
    "name": "Author Name"
  }
  ```

- **Expected Response:**

  - **Success:**

    ```json
    {
      "status": "success",
      "token": "generated token",
      "data": null
    }
    ```

  - **Failure:** An suitable error message will be returned if the token is invalid, expired, already used, the name is empty, or the author is already known.

**b. Display Author** - shows the database's list of authors.

- **Endpoint:** `/author/display`
- **Method:** `GET`
- **Headers:** `Authorization: Bearer <insert generated jwtTokenHere from the users/authenticate>`

- **Expected Response:**

  - **Success:**

    ```json
    {
      "status": "success",
      "token": "generated token",
      "data": null
    }
    ```

  - **Failure:** A suitable error message will be returned if the token has expired, has been used, or is invalid.

**c. Update Author** -updates the database with an author's information.

- **Endpoint:** `/author/update`
- **Method:** `PUT`
- **Headers:** `Authorization: Bearer <insert generated jwtTokenHere from the users/authenticate>`
- **Sample Payload:**

  ```json
  {
  "token": " place your JwtToken Here",
  "authorid": "4",
  "name": "Author Name"
    }
  ```

- **Expected Response:**

  - **Success:**

    ```json
    {
      "status": "success",
      "token": "generated token",
      "data": null 
    }
    ```

  - **Failure:** An suitable error message will be returned if the token has already been used, is invalid or expired, the author ID is not present or cannot be located, or there are no fields to change.

**d. Delete Author** - Deletes an author from the database.

- **Endpoint:** `/author/delete`
- **Method:** `DELETE`
- **Headers:** `Authorization: Bearer <insert generated jwtTokenHere from the users/authenticate>`
- **Sample Payload:**

  ```json
   {
    "token": " place your JwtToken Here",
    "authorid": "4",
    }
  
  ```

- **Expected Response:**

  - **Success:**

    ```json
    {
      "status": "success",
      "Token": "Generated token",
      "data": null
    }
    ```

  - **Failure:** If there are no fields to edit, the author ID is missing or not found, the token has already been used, or it is invalid or expired, the relevant error message will be displayed.

<p align="right">(<a href="#library-management-system">back to top</a>)</p>

<h3 id="book-endpoints">3. Book Endpoints</h3>

**a. Register Book** - Register/add a new book to the library.

- **Endpoint:** `/book/register`
- **Method:** `POST`
- **Headers:** `Authorization: Bearer <insert generated jwtTokenHere from the users/authenticate>`
- **Sample Payload:**

  ```json
  {
    "token": "place your JwtToken Here",
    "title": "Book Title"
    "authorid": "4"
  }
  ```

- **Expected Response:**

  - **Success:**

    ```json
    {
      "status": "success",
      "Token": "generated token",
      "data": null
    }
    ```

  - **Failure:** An appropriate error message will be returned if the token is invalid, expired, already used, the title is empty, or the book already exists.

**b. Display Books** - presents a database list of books.

- **Endpoint:** `/book/display`
- **Method:** `GET`
- **Headers:** `Authorization: Bearer <insert generated jwtTokenHere from the users/authenticate>`

- **Expected Response:**

  - **Success:**

    ```json
    {
      "status": "success",
      "Token": "generated token",
      "data": [
        {
          "bookid": 1,
          "title": "Book Title"
        }
      ]
    }
    ```

  - **Failure:** The relevant error message will be displayed if the token has already been used, is invalid, or has expired.

**c. Update Book** - updates the database's information on a book.

- **Endpoint:** `/book/update`
- **Method:** `PUT`
- **Headers:** `Authorization: Bearer <insert generated jwtTokenHere from the users/authenticate>`
- **Sample Payload:**

  ```json
  {
    "token":" place your JwtToken Here",
    "bookid": 1,
    "title": "Updated Book Title",
    "authorid":"4"
  }
  ```

- **Expected Response:**

  - **Success:**

    ```json
    {
      "status": "success",
      "token": "generated token",
      "data": null
    }
    ```

  - **Failure:** A suitable error message will be supplied if the token has already been used, is invalid or expired, the book ID is missing or cannot be located, or there are no fields to change.

**d. Delete Book** - removing a book from the database..

- **Endpoint:** `/book/delete`
- **Method:** `DELETE`
- **Headers:** `Authorization: Bearer <insert generated jwtTokenHere from the users/authenticate>`
- **Sample Payload:**

  ```json
  {
    "token": "place your JwtToken Here",
    "bookid": 4
  }
  ```

- **Expected Response:**

  - **Success:**

    ```json
    {
      "status": "success",
      "token": "generated token",
      "data": null
    }
    ```

  - **Failure:** A suitable error message will be supplied if the token has already been used, is invalid or expired, or if the book ID cannot be located.

<p align="right">(<a href="#library-management-system">back to top</a>)</p>

<h3 id="book-author-relationship-endpoints">4. Book-Author Relationship Endpoints</h3>

**a. Add Book-Author** - Adds a new association between a book and an author.

- **Endpoint:** `/books_author/add`
- **Method:** `POST`
- **Headers:** `Authorization: Bearer <insert generated jwtTokenHere from the users/authenticate>`
- **Sample Payload:**

  ```json
  {
    "bookid": 1,
    "authorid": 2
  }
  ```

- **Expected Response:**

  - **Success:**

    ```json
    {
      "status": "success",
      "data": null
    }
    ```

  - **Failure:** If the token is already used, invalid/expired, or if required fields (book ID or author ID) are missing, the response will indicate the specific error.

**b. Display All Book-Author** - Displays all book-author associations in the database with their corresponding IDs.

- **Endpoint:** `/books_author/display`
- **Method:** `GET`
- **Headers:** `Authorization: Bearer <insert generated jwtTokenHere from the users/authenticate>`

- **Expected Response:**

  - **Success:**

    ```json
    {
      "status": "success",
      "data": [
        {
          "collectionid": 1,
          "bookid": 1,
          "authorid": 2
        }
      ]
    }
    ```

  - **Failure:** If the token is already used, invalid/expired, or any database issue occurs, the response will indicate the specific error.

**c. Display Book-Author with Names** - Displays book-author associations with the book and author names instead of IDs.

- **Endpoint:** `/books_author/display_with_names`
- **Method:** `GET`
- **Headers:** `Authorization: Bearer <insert generated jwtTokenHere from the users/authenticate>`

- **Expected Response:**

  - **Success:**

    ```json
    {
      "status": "success",
      "data": [
        {
          "collectionid": 1,
          "book_name": "Book Title 1",
          "author_name": "Author Name 1"
        }
      ]
    }
    ```

  - **Failure:** If the token is already used, invalid/expired, or any database issue occurs, the response will indicate the specific error.

**d. Update Book-Author** - Updates an existing book-author association by changing the book and/or author ID.

- **Endpoint:** `/books_author/update`
- **Method:** `PUT`
- **Headers:** `Authorization: Bearer <insert generated jwtTokenHere from the users/authenticate>`
- **Sample Payload:**

  ```json
  {
    "collectionid": 1,
    "bookid": 2,
    "authorid": 3
  }
  ```

- **Expected Response:**

  - **Success:**

    ```json
    {
      "status": "success",
      "data": null
    }
    ```

  - **Failure:** If the token is already used, invalid/expired, if the collection ID is missing or not found, or no fields are provided to update, the response will indicate the specific error.

**e. Delete Book-Author** - Deletes a specific book-author association.

- **Endpoint:** `/books_author/delete`
- **Method:** `DELETE`
- **Headers:** `Authorization: Bearer <insert generated jwtTokenHere from the users/authenticate>`
- **Sample Payload:**

  ```json
  {
    "collectionid": 1
  }
  ```

- **Expected Response:**

  - **Success:**

    ```json
    {
      "status": "success",
      "data": null
    }
    ```

  - **Failure:** If the token is already used, invalid/expired, if the collection ID is missing or no association exists for the given ID, the response will indicate the specific error.

<p align="right">(<a href="#library-management-system">back to top</a>)</p>

## Token Management

**Check if Token is Used**  
The `isTokenUsed` function checks the `used_tokens` table to see if the token has been recorded as used.

```php
function isTokenUsed($token, $conn)
{
    $stmt = $conn->prepare("SELECT * FROM used_tokens WHERE token = :token");
    $stmt->bindParam(':token', $token);
    $stmt->execute();
    return $stmt->rowCount() > 0;
}
```

**Validate Token**  
The `validateToken` function decodes and validates the token using the secret key, returning `false` if the token is invalid or expired.

```php
function validateToken($token, $key)
{
    try {
        return JWT::decode($token, new Key($key, 'HS256'));
    } catch (Exception $e) {
        return false;
    }
}
```

**Mark Token as Used**  
The `markTokenAsUsed` function inserts the token into the `used_tokens` table, marking it as used to prevent reuse.

```php
function markTokenAsUsed($conn, $token)
{
    try {
        $stmt = $conn->prepare("INSERT INTO used_tokens (token) VALUES (:token)");
        $stmt->bindParam(':token', $token);
        $stmt->execute();
    } catch (PDOException $e) {
        throw new Exception("Error marking token as used: " . $e->getMessage());
    }
}
```

<p align="right">(<a href="#library-management-system">back to top</a>)</p>

## Troubleshooting / FAQ

- **Q: How do I regenerate an expired token?**  
  **A:** To regenerate an expired token, you need to log in again by providing valid credentials. The new token will be issued upon successful authentication.

- **Q: Why am I getting a "Token is invalid" error?**  
  **A:** This error occurs if the token is either malformed or has expired. Ensure the token is correctly formatted and that it is not expired. If necessary, regenerate a new token.

- **Q: How do I check if my token has already been used?**  
  **A:** You can check if your token has been used by calling the API endpoint that checks the token's status, or by inspecting the `used_tokens` table in the database.

- **Q: What should I do if I get a "Token expired" error when calling the API?**  
  **A:** This error means the token has expired. Regenerate the token by logging in again, and ensure you are using the new token for your API requests.

- **Q: Can I use the same token multiple times?**  
  **A:** No, the token can only be used once. After it is used, it will be marked as "used" in the database, and any subsequent attempts to reuse it will result in a "Token already used" error.

- **Q: Where will I put the token if I'm using Thunder Client?**  
  **A:**
  - Open Thunder Client and create a new request.
  - Go to the "Headers" tab.
  - Add a new key called `Authorization`.
  - Set the value to `Bearer <your_token>`, where `<your_token>` is the JWT token you want to use.  
    For example:  
    `Authorization: Bearer your_token_here`

<p align="right">(<a href="#library-management-system">back to top</a>)</p>

## Project Information

This project is developed as part of a midterm school requirement for ITPC 115. It is aimed at demonstrating knowledge and skills in building secure API endpoints and token management.

<p align="right">(<a href="#library-management-system">back to top</a>)</p>
