# Artificial Non Intelligence - Data Analysis

A deep learning generated web game to raise awareness about AI and trolls.
This repository is about the Data Analysis and data.

For the API, check the [dedicated API repo](https://github.com/bolinocroustibat/artificial-non-intelligence-api).

For the frontend, check the [dedicated frontend repo](https://github.com/bolinocroustibat/artificial-non-intelligence-frontend).

## Notebooks

These notebooks contain the data analysis and processing work for this project including exploratory data analysis of tweets, testing text generation capabilities, generating fake tweets, and comparing various ML and DL models in their ability to detect fake tweets.     

- [Exploratory Data Analysis & Text Generation for Fake Tweets](https://colab.research.google.com/drive/1ZcTjCIe_RXHtVwk9v_z5SlwRmfdnEBfM?usp=sharing)
- [Model Comparison & Selection to represent AI Player](https://colab.research.google.com/drive/1INR2W0NlG5qYsan94eS-hUvSOmzBO38O?usp=sharing)

Preliminary R&D notebooks: 
- [R&D-A](https://colab.research.google.com/drive/1pjQPQVu6jJOYleQ1VoCv_kRtPeVgc3NR)
- [R&D-GPT2 Primer Text Approach](https://colab.research.google.com/drive/1NHUtwSCIZSj4I8q8KmLBY3SKtP6mTEDB)
- [R&D-RNN Text Generation](https://colab.research.google.com/drive/1Wm9Go9oA6_wQz5gGinOfYJxkUWulvXjL)
- [R&D-Template](https://colab.research.google.com/drive/1FevBBLTL4EByWy49a-EUx5fHmr_z6I2M)
- [R&D-GPT2 Resources](https://colab.research.google.com/drive/1PTbX8Ncl-OiZgqut6X06a4yKZ2roGItF?usp=sharing)

## Main dependencies

- Python 3.9
- MySQL database (there were a previous version using PostgreSQL to be hosted by Heroku, but it's deprecated. Still available on [heroku branch](https://github.com/bolinocroustibat/artificial-non-intelligence-data/tree/heroku))
- Streamlit

Create a virtual environemment for the project, and install the Python dependencies packages with:
```sh
pip install -r requirements.txt
```

...or, if you use [Poetry](https://python-poetry.org/) (which is much better and strongly advised):
```sh
poetry install
```

## Data

JSON files from Kaggle and generated by external Notebooks are located in `/data` folder.


## To run locally

Create or update a settings.py file on the root with the following settings (example):
```
DATABASE_HOST="127.0.0.1"
DATABASE_USER="root"
DATABASE_PASSWORD="root"
DATABASE_PORT=8889
DATABASE_DB="artificial-non-intelligence"
```

```sh
streamlit run app.py
```


## Import data as JSON files into the database

Use the following script and edit it with the right filenames:
`python api/import_json.py`

Please note that it's recommended to first remove the existing comments from the database before importing. To do so, you can use this SQL command:
```sql
DELETE FROM comments;
```
(this should be added to the import script in the future)


## Comments crawlers

Located in /scraper.

Python script to get comments from online website comments and put them in a SQlite database. Uses a Peewee ORM and needs to be adapted to the current DB architecture.

Launch the crawler with Python from typer command file:
```sh
python3 ./scraper/crawl.py [website-crawler]
```

for example, for the crawler "Le Figaro":
```sh
python3 ./scraper/crawl.py figaro
```


## Database

### Schema

The database consists of two tables:

- `comments`: stores the human-generated and ai-generated comments, along with their unique ID, a flag `real` to indicate if it's human or ai generated and a few minor other infos. This table is used for the API to serve the game content.
```sql
CREATE TABLE comments (
	id SERIAL,
	content TEXT NOT NULL,
	real INTEGER NOT NULL,
	aggressive INTEGER,
	difficulty INTEGER,
	created timestamp DEFAULT CURRENT_TIMESTAMP
);
```

- `answers`: stores the answers from users from the game. Each answer has a foreign key to the `comments` table, and the IP adress of the user, along with few minor other infos:
```sql
CREATE TABLE answers (
	id SERIAL,
	answer INT NOT NULL,
	ip VARCHAR,
	comment INTEGER NOT NULL,
	FOREIGN KEY (comment) REFERENCES comments (id)
);
```


### Remove duplicate from the comments table of the DB

The import script doesn't check in the DB if there's any duplicates.
To remove the duplicated comments content from the DB at any time, use the following SQL command:
```sql
DELETE FROM comments
WHERE id NOT in(
		SELECT
			min(id)
			FROM comments
		GROUP BY
			content);
```
(this should be added to the import script in the future)
