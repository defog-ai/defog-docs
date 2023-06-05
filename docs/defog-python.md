---
sidebar_position: 2
---

# Using the Python Library

Once you have set up defog using `defog init` from the command line, you can start using it in your applications.

## Initializing defog in your Python app
Initializing Defog is as simple as

```python
from defog import Defog

# uses the connection details you used 
# when running defog init
defog = Defog()
```

If you want to initialize with a different set of database credentials, you can use the following

```python

from defog import Defog
defog = Defog(
    api_key = "YOUR_DEFOG_API_KEY",
    db_type = "YOUR_DB_TYPE",
    # must be one of postgres, redshift, mysql, bigquery,
    # mongo, or snowflake
    db_creds = YOUR_DB_CREDS
    # must be a dict in these formats, depending
    # on your database type
    # https://github.com/defog-ai/defog-python/blob/63af5e3ded07da356365f20bc94a194c4f7c44fa/defog/__init__.py#L110
)
```

## Using Defog to answer questions

### To automatically convert a query to SQL, and run it on your DB
```python
defog = Defog()
results = defog.run_query("how many users do we have?")
results
{
    'columns': ['num_users'],
    'data': [(49990,)],
    'query_generated': 'SELECT COUNT(userid) AS num_users FROM users;',
    'ran_successfully': True,
    'reason_for_query': 'The users table contains a column called userid which is a unique identifier for each user. To count the number of users, we can simply use the COUNT() function on the userid column in the users table.',
    'previous_context': [
        'how many users do we have?',
        'SELECT COUNT(userid) AS num_users FROM users;'
    ]
}
```

Optionally, you can also pass the list in `previous_context` to other functions, so that the model can adapt to your question.

```python
defog = Defog()
defog.run_query(
    "how many from San Francisco",
    previous_context=results['previous_context']
)
{
    'columns': ['num_users'],
    'data': [(50,)],
    'query_generated': "SELECT COUNT(*) AS num_users FROM users WHERE city ILIKE '%San Francisco%';",
    'ran_successfully': True,
    'reason_for_query': "The user is asking for the number of users from San Francisco. The city of the user is stored in the 'city' column of the 'users' table. Therefore, we can use a simple COUNT query to count the number of users from San Francisco. We will use the ILIKE operator to perform a case-insensitive match on the city name, as the user may have typed it in different ways. Since the query only requires one table, we do not need to use a JOIN statement. ",
    'previous_context': [
        'how many users do we have?',
        'SELECT COUNT(userid) AS num_users FROM users;',
        'how many from San Francisco?',
        "SELECT COUNT(*) AS num_users FROM users WHERE city ILIKE '%San Francisco%';"
    ]
}

```

### Adding hard filters for secure data access
If you want to restrict which results a user can see, you can just add the `hard_filters` property to the `run_query` function.

```python
defog = Defog()
defog.run_query(
    "how many users do we have?",
    hard_filters=" Only include data for the venueid 220, and no other venue",
)
{
    'columns': ['count'],
    'data': [(1622,)],
    'query_generated': 'SELECT COUNT(DISTINCT userid) \nFROM users \nJOIN sales ON users.userid = sales.buyerid \nJOIN event ON sales.eventid = event.eventid \nWHERE event.venueid = 220;',
    'ran_successfully': True,
    'reason_for_query': 'The query asks for the total number of users in the database. The relevant table for this query is the users table, which contains a unique identifier for each user. We can simply count the number of distinct user IDs in the table to get the total number of users. We also need to filter the results to only include data for the venueid 220, as specified in the question. Therefore, the SQL query to answer this question is: ', 'previous_context': [
        'how many users do we have?',
        'SELECT COUNT(DISTINCT userid) FROM users JOIN event ON users.userid = event.buyerid WHERE event.venueid = 220;'
    ]
}
```

## A minimal webserver running Defog
You can spin up a minimal webserver for testing Defog using Flask.

```python
from flask import Flask, request, jsonify
from flask.json import JSONEncoder
from defog import Defog
import decimal

class JsonEncoder(JSONEncoder):
    # for handling decimals
    def default(self, obj):
        if isinstance(obj, decimal.Decimal):
            return float(obj)
        return JSONEncoder.default(self, obj)

defog = Defog()

app = Flask(__name__)
app.json_provider_class = JsonEncoder

@app.route("/", methods=['POST'])
def test_defog():
    data = request.json
    question = data.get('question')
    previous_context = data.get('previous_context')
    results = defog.run_query(question=question, previous_context=previous_context)
    return jsonify(results)

if __name__ == "__main__":
    app.run(debug=True, host='localhost', port=5000)
```

While this is running, you can test that it works by making a POST request to localhost:5000, like this

```python
import requests
r = requests.post("http://localhost:5000/", json={"question": "how many users to we have?"})
r.json()
```

Alternatively, you can also paste the link `http://localhost:5000/` as an `apiEndpoint` in your [React component](/defog-react) and test it locally.