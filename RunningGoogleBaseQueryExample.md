# Introduction #

These instructions will guide you through running the example script which performs queries against the Google Base snippets feed.

# Details #

## Execution Environment ##

Make sure that the required modules are in your Python Path. See DependancyModules for a list of things you may need to install.

## Running the example ##

You can run the example from the main project directory by executing `python samples/base/baseQueryExample.py`

You will be prompted for a query. Try typing `digital camera` to see titles for items which match the query `bq=digital+camera`. The script will show the titles for the first 10 search results, and then prompt you to press enter for the next 10. After displaying around 1,000 items, the script will exit.

You may exit the application at any time by terminating the application using control-D or control-C.

## Example Queries ##

Here are some example queries you may like to try to get a feel for how the Google Base query language works. If you would like the example code to display the query URL you can add `print q.ToUri()` in the while loop.

```
digital camera
[item type: digital camera]
```