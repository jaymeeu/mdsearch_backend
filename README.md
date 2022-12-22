### What i built

I built a beckend with NodeJS(Express) and MongoDB. I implemented the item search with autocomplete feature with MongoDB Atlas.
To see these feature in action checkout the live link below

### App Link
https://windcnc.netlify.app/

### Link to Source Code
[Source code frontend](https://github.com/jaymeeu/mdsearch_no_more)

[Source code backend](https://github.com/jaymeeu/mdsearch_backend)

### Get started
After cloning or forking, run the below command
```bash
    npx nodemon .
```

### How i built it
The following are steps to implement the project

#### Connecting to mongoDB collection

Our collection name `sample_airbnb`

```javascript
try {
    const client = await MongoClient.connect(MONGODB_URI, { useNewUrlParser: true, useUnifiedTopology: true });
    db = await client.db("sample_airbnb");
    dbConnected = true;
    
    console.log("Database is connected and ready to go!");

  } catch (e) {
    console.log(e.toString(), "connection error");
  }
```

#### Create search indexes


![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/hx4iwq95mtat43pxnf39.png)

```javascript
{
  "mappings": {
    "dynamic": false,
    "fields": {
      "address": {
        "dynamic": false,
        "fields": {
          "country": {
            "type": "string"
          },
          "street": {
            "type": "string"
          }
        },
        "type": "document"
      },
      "property_type": {
        "type": "string"
      }
    }
  }
}
```
#### Create search indexes for country autocomplete
```javascript
{
  "mappings": {
    "dynamic": false,
    "fields": {
      "address": {
        "fields": {
          "country": {
            "maxGrams": 10,
            "type": "autocomplete"
          }
        },
        "type": "document"
      }
    }
  }
}
```
#### Create search indexes for street autocomplete
```javascript
{
  "mappings": {
    "dynamic": false,
    "fields": {
      "address": {
        "fields": {
          "street": {
            "type": "autocomplete"
          }
        },
        "type": "document"
      }
    }
  }
}
```

#### Create aggregation pipeline to get list on homes in a category `property_type`
```javascript
 [
        {
          '$match': {
            'property_type': {
              '$eq': req.params.category
            }
          }
        }, {
          '$project': {
            'accommodates': 1,
            'price': 1,
            'property_type': 1,
            'name': 1,
            'description': 1,
            'host': 1,
            'address': 1,
            'images': 1,
            "review_scores": 1
          }
        }
      ]
```
#### Create aggregation pipeline to search list of homes by country in each category
```javascript
[ 
{
    "$search": {
      "index": 'search_home',
      "compound": {
        "must": [
        {"text": {
          "query": queries.street,
          "path": 'address.street',
          "fuzzy": {}
        }},
        { "text": {
          "query": queries.category,
          "path": 'property_type',
          "fuzzy": {}
        }}
      ]}
    }
  }
]
```
#### Create aggregation pipeline to search list of homes by street in each caterogy
```javascript
[{
    "$search": {
      "index": 'search_home',
      "compound": {
        "must": [
       { "text": {
          "query": queries.country,
          "path": 'address.country',
          "fuzzy": {}
        }},
        { "text": {
          "query": queries.category,
          "path": 'property_type',
          "fuzzy": {}
        }}
      ]}
    }
  }]
```
#### Create aggregation pipeline to search list on homes by both country and street in each category
```javascript
[{
    "$search": {
      "index": 'search_home',
      "compound": {
        "must": [
        {"text": {
          "query": queries.street,
          "path": 'address.street',
          "fuzzy": {}
        }},
       { "text": {
          "query": queries.country,
          "path": 'address.country',
          "fuzzy": {}
        }},
        { "text": {
          "query": queries.category,
          "path": 'property_type',
          "fuzzy": {}
        }}
      ]}
    }
  }]
```

#### Create aggregation pipeline for autocomplete country search field
```javascript
[
        {
          '$search': {
            'index': 'country_autocomplete', 
            'autocomplete': {
              'query': req.params.param, 
              'path': 'address.country',
            }, 
            'highlight': {
              'path': [
                'address.country'
              ]
            }
          }
        }, {
          '$limit': 1
        }, {
          '$project': {
            'address.country': 1, 
            'highlights': {
              '$meta': 'searchHighlights'
            }
          }
        }
      ]
```
#### Create aggregation pipeline for autocomplete street search field
```javascript
[
        {
          '$search': {
            'index': 'town_autocomplete', 
            'autocomplete': {
              'query': req.params.param, 
              'path': 'address.street',
            }, 
            'highlight': {
              'path': [
                'address.street'
              ]
            }
          }
        }, {
          '$limit': 5
        }, {
          '$project': {
            'address.street': 1, 
            'highlights': {
              '$meta': 'searchHighlights'
            }
          }
        }
      ]
```
