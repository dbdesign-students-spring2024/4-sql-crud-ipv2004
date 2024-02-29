# SQL CRUD

An assignment to design relational database tables with particular applications in mind.

The contents of this file will be deleted and replaced with the content described in the [instructions](./instructions.md)
## Part 1
### 1. Create the table to store the restaraunt names and their information
```sql
CREATE TABLE restaurants (
restaurant_id INTEGER PRIMARY KEY AUTOINCREMENT,
name TEXT NOT NULL,
category TEXT,
price_tier TEXT CHECK(price_tier IN ('Cheap', 'Medium', 'Expensive')) NOT NULL,
neighborhood TEXT,
opening_hours TEXT,
closing_hours TEXT,
good_for_kids TEXT CHECK(good_for_kids IN ('TRUE', 'FALSE'))
;
```
### 2. Create table to store the review numbers 
```sql
CREATE TABLE reviews (
review_id INTEGER PRIMARY KEY AUTOINCREMENT,
restaurant_id INTEGER,
user_id INTEGER,
review_text TEXT,
rating INTEGER CHECK(rating >= 1 AND rating <= 5),
FOREIGN KEY (restaurant_id) REFERENCES restaurants(restaurant_id)
);
```
### 3. Links to the Mock Data
[Restaurant Data](data/restaurants.csv)
[Review Data](data/MOCK_DATAreviews.csv)
### 4. Import code
```sql
.mode csv
.import C:\Users\ishar\Downloads\MOCK_DATA.csv restaurants
.import C:\Users\ishar\Downloads\MOCK_DATA.csv restaurants
```
### 5.  Queries
```sql
a. **Find all cheap restaurants in a particular neighborhood (pick any neighborhood as an example).**
SELECT * FROM restaurants
WHERE price_tier = 'Cheap' AND neighborhood = 'Upper West Side';
```
b. **Find all restaurants in a particular genre (pick any genre as an example) with 3 stars or more, ordered by the number of stars in descending order.**
```sql
SELECT r.restaurant_id, r.name, r.category, AVG(re.rating) AS average_rating
FROM restaurants r
JOIN reviews re ON r.restaurant_id = re.restaurant_id
WHERE r.category = 'Thai'
GROUP BY r.restaurant_id, r.name, r.category
HAVING AVG(re.rating) >= 3
ORDER BY average_rating DESC;
```
c. **Find all restaurants that are open now (see hint below).**
```sql
SELECT *
FROM restaurants
WHERE strftime('%H:%M', 'now', 'localtime') >= opening_hours
AND strftime('%H:%M', 'now', 'localtime') < closing_hours;
```
d. **Leave a review for a restaurant (pick any restaurant as an example; note that leaving a review has no automatic effect on the average rating of the restaurant).**
```sql
INSERT INTO reviews (restaurant_id, rating, review_text, review_date)
VALUES (1, 5, 'The food was bussin');
```
e. **Delete all restaurants that are not good for kids.**
```sql
DELETE FROM restaurants
WHERE good_for_kids = 'FALSE';
```
f. **Find the number of restaurants in each NYC neighborhood.**
```sql
SELECT neighborhood, COUNT(*) AS number_of_restaurants
FROM restaurants
GROUP BY neighborhood
ORDER BY number_of_restaurants;
```

## Part 2
### 1. Creating 4 tables to store users, posts, and messages and stories sepertely:
```sql
CREATE TABLE users (
    user_id INTEGER PRIMARY KEY AUTOINCREMENT,
    email TEXT UNIQUE NOT NULL,
    password TEXT NOT NULL,
    handle TEXT UNIQUE NOT NULL
);
CREATE TABLE posts (
    post_id INTEGER PRIMARY KEY AUTOINCREMENT,
    user_id INTEGER NOT NULL,
    post_type TEXT CHECK(post_type IN ('message', 'story')) NOT NULL,
    created_at DATETIME DEFAULT (datetime('now')),
    FOREIGN KEY (user_id) REFERENCES users(user_id)
);
CREATE TABLE messages (
    message_id INTEGER PRIMARY KEY,
    recipient_id INTEGER NOT NULL,
    content TEXT NOT NULL,
    FOREIGN KEY (message_id) REFERENCES posts(post_id),
    FOREIGN KEY (recipient_id) REFERENCES users(user_id)
);
CREATE TABLE stories (
    story_id INTEGER PRIMARY KEY,
    content TEXT NOT NULL,
    FOREIGN KEY (story_id) REFERENCES posts(post_id)
);
```
### 2. Links to the mock data
[User Data](data/appmockdata.csv)
[Post Data](data/posts.csv)
[Story Data](data/stories.csv)
[Messages Data](data/messages.csv)
### 3. Import Code
.import C:\Users\ishar\Downloads\appmockdata.csv users
.import C:\Users\ishar\Downloads\posts.csv posts
.import C:\Users\ishar\Downloads\messages.csv messages
.import C:\Users\ishar\Downloads\stories.csv stories
### 4. Queries
a. **Register a new User.**
INSERT INTO users (email, password, handle) VALUES ('user@example.com', 'password123', 'userhandle');
b. **Create a new Message sent by a particular User to a particular User (pick any two Users for example).**
INSERT INTO posts (user_id, post_type, created_at) VALUES (1, 'message', datetime('now'));
INSERT INTO messages (message_id, recipient_id, content) VALUES (last_insert_rowid(), 2, 'Hello from User 1 to User 2');
c. **Create a new Story by a particular User (pick any User for example).**
INSERT INTO posts (user_id, post_type, created_at) VALUES (1, 'story', datetime('now'));
INSERT INTO stories (story_id, content) VALUES (last_insert_rowid(), 'Story by User 1');
d. **Show the 10 most recent visible Messages and Stories, in order of recency.**
SELECT p.post_id, p.post_type, p.created_at, IFNULL(m.content, s.content) AS content
FROM posts p
LEFT JOIN messages m ON p.post_id = m.message_id
LEFT JOIN stories s ON p.post_id = s.story_id
ORDER BY p.created_at DESC
LIMIT 10;
e. **Show the 10 most recent visible Messages sent by a particular User to a particular User (pick any two Users for example), in order of recency.**
SELECT m.content
FROM messages m
JOIN posts p ON m.message_id = p.post_id
WHERE p.user_id = 1 AND m.recipient_id = 2
ORDER BY p.created_at DESC
LIMIT 10;
f. **Make all Stories that are more than 24 hours old invisible.**
SELECT story_id
FROM stories s
JOIN posts p ON s.story_id = p.post_id
WHERE (JULIANDAY('now') - JULIANDAY(p.created_at)) * 24 > 24;
g. **Show all invisible Messages and Stories, in order of recency.**
-- Placeholder query; adjust based on your invisibility mechanism
SELECT post_id, post_type, created_at
FROM posts
WHERE invisible = 1 -- Assuming there's an 'invisible' flag
ORDER BY created_at DESC;
h. **Show the number of posts by each User.**
SELECT user_id, COUNT(*) as total_posts
FROM posts
GROUP BY user_id;
i. **Show the post text and email address of all posts and the User who made them within the last 24 hours.**
SELECT u.email, IFNULL(m.content, s.content) AS content
FROM posts p
LEFT JOIN users u ON p.user_id = u.user_id
LEFT JOIN messages m ON p.post_id = m.message_id
LEFT JOIN stories s ON p.post_id = s.story_id
WHERE p.created_at >= datetime('now', '-1 day');
j. **Show the email addresses of all Users who have not posted anything yet.**
SELECT email
FROM users
WHERE user_id NOT IN (SELECT DISTINCT user_id FROM posts);




