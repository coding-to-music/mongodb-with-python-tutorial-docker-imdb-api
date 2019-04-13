# mongodb-with-python-tutorial
MongoDB with Pymongo Tutorial

### Installing

```
$ pip install pymongo
```

### Making a Connection

Making a connection without authentication:

```
>>> from pymongo import MongoClient
>>> uri = 'mongodb://mongodb.domain.com:27017/'
>>> client = MongoClient(uri)
>>> client.database_names()
[u'admin', u'local', u'mycargarage', u'random_api', u'shared_db', u'testdb']
```

Making a connection with authentication:

```
>>> from pymongo import MongoClient
>>> uri = 'mongodb://username:password@mongodb.domain.com:27017/random_api?authSource=admin&authMechanism=SCRAM-SHA-1'
>>> client = MongoClient(uri)
>>> client.database_names()
[u'admin', u'local', u'mycargarage', u'random_api', u'shared_db', u'testdb']
```

### Listing Databases


```
>>> client = MongoClient(uri)
>>> client.database_names()
[u'admin', u'local', u'mycargarage', u'random_api', u'shared_db', u'testdb']
```

### Listing Collections

```
>>> client = MongoClient(uri)
>>> db = client.shared_db
>>> db.collection_names()
[u'flask_reminders', u'test', u'usersessions', u'messages']
```

### Write One Document 

Create a database `store_db`, use the `transactions` collection and write a document to it.

```
>>> db = client.store_db
>>> transactions = db.transactions
>>> doc_data = {
    'store_name': 'sportsmans', 
    'branch_name': 'tygervalley', 
    'account_id': 'sns_03821023', 
    'total_costs': 109.20, 
    'products_purchased': ['cricket bat', 'cricket ball', 'sports hat'], 
    'purchase_method': 
    'credit card'
}
>>> response = transactions.insert_one(doc_data)
>>> response.inserted_id
ObjectId('5cad16a5a5f3826f6f046d74')
```

We can verify that the collection is present:

```
>>> db.collection_names()
[u'transactions']
```

### Write Many Documents

We can batch up our writes:

```
>>> transaction_1 = {
    'store_name': 'sportsmans', 'branch_name': 'tygervalley', 
    'account_id': 'sns_09121024', 'total_costs': 129.84, 
    'products_purchased': ['sportsdrink', 'sunglasses', 'sports illustrated'], 
    'purchase_method': 'credit card'
}
>>> transaction_2 = {
    'store_name': 'burger king', 'branch_name': 
    'somerset west', 'account_id': 'bk_29151823', 
    'total_costs': 89.99, 'products_purchased': ['cheese burger', 'pepsi'], 
    'purchase_method': 'cash'
}
>>> transaction_3 = {
    'store_name': 'game', 'branch_name': 'bellvile', 'account_id': 'gm_49121229', 
    'total_costs': 499.99, 'products_purchased': ['ps4 remote'], 
    'purchase_method': 'cash'
}
>>> response = transactions.insert_many([transaction_1, transaction_2, transaction_3])
>>> response.inserted_ids
[ObjectId('5cad18d4a5f3826f6f046d75'), ObjectId('5cad18d4a5f3826f6f046d76'), ObjectId('5cad18d4a5f3826f6f046d77')]
```

### Find One Document:

```
>>> transactions.find_one({'account_id': 'gm_49121229'})
{u'account_id': u'gm_49121229', u'store_name': u'game', u'purchase_method': u'cash', u'branch_name': u'bellvile', u'products_purchased': [u'ps4 remote'], u'_id': ObjectId('5cad18d4a5f3826f6f046d77'), u'total_costs': 499.99}
```

### Find Many Documents:

```
>>> response = transactions.find({'purchase_method': 'cash'})
>>> [doc for doc in response]
[{u'account_id': u'bk_29151823', u'store_name': u'burger king', u'purchase_method': u'cash', u'branch_name': u'somerset west', u'products_purchased': [u'cheese burger', u'pepsi'], u'_id': ObjectId('5cad18d4a5f3826f6f046d76'), u'total_costs': 89.99}, {u'account_id': u'gm_49121229', u'store_name': u'game', u'purchase_method': u'cash', u'branch_name': u'bellvile', u'products_purchased': [u'ps4 remote'], u'_id': ObjectId('5cad18d4a5f3826f6f046d77'), u'total_costs': 499.99}]
```

Or filtering down the results to only the account id:

```
>>> response = transactions.find({'purchase_method': 'cash'})
>>> [doc['account_id'] for doc in response]
[u'bk_29151823', u'gm_49121229']
```

MongoDB also has a count method:

```
>>> transactions.find({'purchase_method': 'cash'}).count()
2
```

Query with the AND condition. SQL equivalent: `where branch_name = 'tygervalley' AND account_id = 'sns_03821023'`

```
>>> response = transactions.find({ '$and': [{'branch_name': 'tygervalley'},{'account_id': 'sns_03821023'}]})
>>> [v for v in response]
[{u'account_id': u'sns_03821023', u'store_name': u'sportsmans', u'purchase_method': u'credit card', u'branch_name': u'tygervalley', u'products_purchased': [u'cricket bat', u'cricket ball', u'sports hat'], u'_id': ObjectId('5cb18881df585e003c976d5d'), u'total_costs': 109.2}]
```

Query with the OR condition. SQL equivalent: `where branch_name = 'tygervalley' OR account_id = 'sns_03821023'`:

```
>>> response = transactions.find({ '$or': [{'branch_name': 'tygervalley'},{'account_id': 'sns_03821023'}]})
>>> [v for v in response]
[{u'account_id': u'sns_03821023', u'store_name': u'sportsmans', u'purchase_method': u'credit card', u'branch_name': u'tygervalley', u'products_purchased': [u'cricket bat', u'cricket ball', u'sports hat'], u'_id': ObjectId('5cb18881df585e003c976d5d'), u'total_costs': 109.2}, {u'account_id': u'sns_09121024', u'store_name': u'sportsmans', u'purchase_method': u'credit card', u'branch_name': u'tygervalley', u'products_purchased': [u'sportsdrink', u'sunglasses', u'sports illustrated'], u'_id': ObjectId('5cb188a3df585e003c976d5e'), u'total_costs': 129.84}]
```

Combining it:

```
>>> response = transactions.find({ 'total_costs': {'$gt': 120}, '$or': [{'branch_name': 'tygervalley'},{'account_id': 'sns_03821023'}]})
>>> [v for v in response]
[{u'account_id': u'sns_09121024', u'store_name': u'sportsmans', u'purchase_method': u'credit card', u'branch_name': u'tygervalley', u'products_purchased': [u'sportsdrink', u'sunglasses', u'sports illustrated'], u'_id': ObjectId('5cb188a3df585e003c976d5e'), u'total_costs': 129.84}]
```

Other condition operations include:

```
lt  - Less Than
lte - Less Than Equals
gt  - Greater Than
gte - Greater Than Equals
ne  - Not Equals
```

### Filters

Find all the documents with purchase price > 120:

```
>>> response = transactions.find({'total_costs': {'$gt': 120}})
>>> [doc for doc in response]
[{u'account_id': u'sns_09121024', u'store_name': u'sportsmans', u'purchase_method': u'credit card', u'branch_name': u'tygervalley', u'products_purchased': [u'sportsdrink', u'sunglasses', u'sports illustrated'], u'_id': ObjectId('5cad18d4a5f3826f6f046d75'), u'total_costs': 129.84}, {u'account_id': u'gm_49121229', u'store_name': u'game', u'purchase_method': u'cash', u'branch_name': u'bellvile', u'products_purchased': [u'ps4 remote'], u'_id': ObjectId('5cad18d4a5f3826f6f046d77'), u'total_costs': 499.99}]
```

### Projections

Select specific fields from the returned response:

```
>>> response = transactions.find({}, {'branch_name': 'tygervalley', 'purchase_method': 'credit card'})
>>> [doc for doc in response]
[{u'purchase_method': u'credit card', u'branch_name': u'tygervalley', u'_id': ObjectId('5cad16a5a5f3826f6f046d74')}, {u'purchase_method': u'credit card', u'branch_name': u'tygervalley', u'_id': ObjectId('5cad18d4a5f3826f6f046d75')}, {u'purchase_method': u'cash', u'branch_name': u'somerset west', u'_id': ObjectId('5cad18d4a5f3826f6f046d76')}, {u'purchase_method': u'cash', u'branch_name': u'bellvile', u'_id': ObjectId('5cad18d4a5f3826f6f046d77')}]
```

### Sorting Documents

Sorting Documents in Descending Order:

```
>>> from pymongo import MongoClient, DESCENDING
>>> response = transactions.find().sort('total_costs', DESCENDING)
>>> ['Products: {}, Price: {}'.format(doc['products_purchased'], doc['total_costs']) for doc in response]
["Products: [u'ps4 remote'], Price: 499.99", "Products: [u'sportsdrink', u'sunglasses', u'sports illustrated'], Price: 129.84", "Products: [u'cricket bat', u'cricket ball', u'sports hat'], Price: 109.2", "Products: [u'cheese burger', u'pepsi'], Price: 89.99"]
```

### Aggregations

```
>>> agr = [{'$group': {'_id': 1, 'all': { '$sum': '$total_costs' }}}]
>> [a for a in transactions.aggregate(agr)]
[{u'all': 829.02, u'_id': 1}]
```

or:

```
>>> agr = [{'$group': {'_id': 1, 'all': { '$sum': '$total_costs' }}}]
>>> val = list(transactions.aggregate(agr))
>>> val
[{u'all': 829.02, u'_id': 1}]
```

Select fields to aggregate, eg. aggregate the costs for selected stores:

```
>>> agr = [{ '$match': {'$or': [ { 'store_name': 'sportsmans' }, { 'store_name': 'game' }] }}, { '$group': {'_id': 1, 'sum2stores': { '$sum': '$total_costs' } }}]
>>> [a for a in transactions.aggregate(agr)]
[{u'_id': 1, u'sum2stores': 739.03}]
```

### Limit Data Output

```
>>> response = transactions.find()
>>> [a['account_id'] for a in response]
[u'sns_03821023', u'sns_09121024', u'bk_29151823', u'gm_49121229']

>>> response = transactions.find().skip(1).limit(3)
>>> [a['account_id'] for a in response]
[u'sns_09121024', u'bk_29151823', u'gm_49121229']

>>> response = transactions.find().skip(1).limit(2)
>>> [a['account_id'] for a in response]
[u'sns_09121024', u'bk_29151823']

>>> response = transactions.find().skip(3).limit(1)
>>> [a['account_id'] for a in response]
[u'gm_49121229']
```

### Drop Collections

```
>>> transactions.drop()
>>> db.collection_names()
[]
```

### MongoEngine - ORM

```
>>> from mongoengine import *
>>> connect('project1', host="mongodb://user:pass@mongodb.domain.com:27017/random_api?authSource=admin&authMechanism=SCRAM-SHA-1")
MongoClient(host=['mongodb.domain.com:27017'], document_class=dict, tz_aware=False, connect=True, read_preference=Primary(), authsource='admin', authmechanism='SCRAM-SHA-1')
```

```
>>> class Student(Document):
    name = StringField(required=True, max_length=200)
    city = StringField(required=True, max_length=200)
    can_code = BooleanField(required=True)

```

Create a student:

```
>>> doc_1 = Student(name='Josh', city='Cape Town', can_code=True)
>>> doc_1.save()
<Student: Student object>
>>> doc_1.id
ObjectId('5cad27dea5f38276a40f43db')
>>> doc_1.name
u'Josh'
>>> doc_1.city
u'Cape Town'
>>> doc_1.can_code
True
```

Test out validation:

```
>>> doc_2 = Student(name='Max', city='Cape Town')
>>> doc_2.save()
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "/usr/local/lib/python2.7/site-packages/mongoengine/document.py", line 369, in save
    self.validate(clean=clean)
  File "/usr/local/lib/python2.7/site-packages/mongoengine/base/document.py", line 392, in validate
    raise ValidationError(message, errors=errors)
mongoengine.errors.ValidationError: ValidationError (Student:None) (Field is required: ['can_code'])
```

Update a user's city:

```
>>> doc_2 = Student(name='Max', city='Cape Town', can_code=False)
>>> doc_2.save()
<Student: Student object>
>>> doc_2.id
ObjectId('5cad2835a5f38276a40f43dc')
>>> doc_2.city
u'Cape Town'
>>> doc_2.city = 'Johannesburg'
>>> doc_2.save()
<Student: Student object>
>>> doc_2.city
'Johannesburg'
```

## Datasets:
- https://raw.githubusercontent.com/steveren/docs-assets/charts-tutorial/movieDetails.json


## Resources

- https://realpython.com/introduction-to-mongodb-and-python/
- https://docs.mongodb.com/charts/master/tutorial/movie-details/prereqs-and-import-data/
