# web_scraping
Code and data for the URAP team that scrapes and parses charter websites using Python, Scrapy, and Beautiful Soup. The data scraped is saved into MongoDB and/or in local files.

The most recent web scraper is `scrapy/schools/schools/spiders/scrapy_vanilla.py`.


## Running the web scraper

### Method 1: install and run locally

If not yet installed, [install MongoDB](https://docs.mongodb.com/manual/installation/).

Navigate to the `/web_scraping/scrapy/schools` directory.
Then run the following commands:
```bash
# Create and start a virtual environment for Python dependencies.
python3 -m venv .venv
source .venv/bin/activate
# Install dependencies.
pip3 install -r requirements.txt
```
If you want to store results into MongoDB, ensure that:

```python
'schools.pipelines.MongoDBPipeline': 3
```
is one of the key-value pairs of `ITEM_PIPELINES` in `/web_scraping/scrapy/schools/schools/settings.py`. If you don't want to use the pipeline, remove the element.

Furthermore, ensure that in `/web_scraping/scrapy/schools/schools/settings.py`, that:
```python
# This next line must NOT be commented.
MONGO_URI = 'mongodb://localhost' 
# This next line is commented.
# MONGO_URI = 'mongodb://mongodb_container:27017'
```
Start MongoDB. This step depends on the operating system. On Ubuntu 18.04, this is:
```bash
sudo systemctl -l start mongodb
```

Finally to run, navigate to `/web_scraping/scrapy/schools/schools/` and run:

```bash
scrapy crawl schoolspider -a csv_input=spiders/test_urls.csv -o schoolspider_output.json
```

This line means to run the schoolspider crawler with the given csv input file
and append the output to a json file. Note that subsequent runs of the crawler will append output, rather than replace. 
Appending to a json file is optional and the crawler can be run without doing this. Not specified in this command, is that data is saved behind the scenes
to a MongoDB database named "schoolSpider" if the MongoDB pipeline is used.


When you're finished and you don't need to run the scraper anymore run:
```
# Deactivate the environment.
deactivate
```

### Method 2: install and run in a container.

Firstly, [get Docker](https://docs.docker.com/get-docker/).

If you want to use MongoDB, ensure that in `/web_scraping/scrapy/schools/schools/settings.py`, that:
```python
# This next line is commented.
# MONGO_URI = 'mongodb://localhost' 
# This next line must NOT be commented.
MONGO_URI = 'mongodb://mongodb_container:27017'
```
And, that:
```python
'schools.pipelines.MongoDBPipeline': 3
```
is one of the key-value pairs in of `ITEM_PIPELINES` in `/web_scraping/scrapy/schools/schools/settings.py`. If you don't want to use the pipeline, remove the element.

Finally, inside `/web_scraping`, run:
```bash
# build the containers and run in the background
docker-compose up --build -d 
# to shutdown when finished
docker-compose down
```
No output json file is created through this method. Rather, the primary method of storing data is through MongoDB
(if it's enabled). Data inside MongoDB is persisted through a volume defined in `docker-compose.yml`.




## Legacy comments

Currently, the main file being used to analyze the data is **scripts/WebScraper.py**. However, this file can take a reasonable amount of time
 for large data sets, **scripts/ScrapyWebScraper** and **scripts/selenium_spider** are being used to speed up the scraping process.  The main file uses selenium and beautiful soup to 
visit a hompage for each school, gather all of the links that the hompage points to, and scrapes the text off of each link. 
Each time the program is run, the results directory is updated.
The diagnostics folder is where data about the time it took to run the program, 
and the accuracy of the program is stored.
