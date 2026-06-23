# Database Design

Database Engine: SQLite

---

# ERD Overview

games
|
+---- launches
|
+---- favorites
|
+---- screenshots
|
+---- game_tags

---

# Table: games

CREATE TABLE games (

id INTEGER PRIMARY KEY,

name TEXT NOT NULL,

slug TEXT UNIQUE,

genre TEXT,

developer TEXT,

publisher TEXT,

description TEXT,

cover_path TEXT,

exe_path TEXT,

install_path TEXT,

size_gb REAL,

status TEXT,

date_added DATETIME

);

---

# Table: launches

CREATE TABLE launches (

id INTEGER PRIMARY KEY,

game_id INTEGER,

launch_time DATETIME,

duration_seconds INTEGER

);

---

# Table: favorites

CREATE TABLE favorites (

id INTEGER PRIMARY KEY,

game_id INTEGER

);

---

# Table: screenshots

CREATE TABLE screenshots (

id INTEGER PRIMARY KEY,

game_id INTEGER,

image_path TEXT

);

---

# Table: tags

CREATE TABLE tags (

id INTEGER PRIMARY KEY,

name TEXT

);

---

# Table: game_tags

CREATE TABLE game_tags (

game_id INTEGER,

tag_id INTEGER

);

---

# Index Strategy

CREATE INDEX idx_game_name
ON games(name);

CREATE INDEX idx_launch_time
ON launches(launch_time);

CREATE INDEX idx_launch_game
ON launches(game_id);

---

# Future Tables

collections

recommendations

analytics_daily

analytics_monthly
