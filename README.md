# Using UnifyQL To Query Data

Works with the PyUnify (Python) and UnifyJS (Javascript) libraries

## UPDATE

The following query will update one cell:
```
unify.query('meta', 'update', [{'row':9, 'col':'value', 'set':"Yahoo Finance Download"}])
```

The following query will update two cells:
```
unify.query('meta', 'update', [{'row':9, 'col':'value', 'set':"Yahoo Finance Download"}, {'row':6, 'col':'value', 'set':'yahoo.com'}])
```

## SELECT

The following query:
```
unify.query('meta', 'select', {'col':'key', 'target':'contract'})
```

Returns:
```
unify.query('meta', 'select', {'col':'key', 'target':'contract'})
```

## INSERT

The following query inserts one row:

```
unify.query('data', 'insert', [{'Animal':'Unicorn', 'Size':'Very Large', 'Gender':None, 'Safe As Pet':True, 'Weight In Pounds':1040.2}])
```
Returns:
```
Row number 5 added
```

The following query inserts two rows:
```
unify.query('data', 'insert', [{'Animal':'Unicorn', 'Size':'Very Large', 'Gender':None, 'Safe As Pet':True, 'Weight In Pounds':1040.2}, {'Animal':'T-Rex', 'Size':'Very, Very Enormous', 'Gender':'Female', 'Safe As Pet':False, 'Weight In Pounds':92283.23}])
```
Returns:
```
Row number 5 added
Row number 6 added
```

## DELETE

The following query will delete the following rows:
```
unify.query('meta', 'delete', [{"rows":[4, 5, 6, 8, 9]}])
```

## PyUnify_Query
```
class PyUnify_SQL:
  def __init__(self, json_unify, data_list):    
    self.json_unify = json_unify
    self.data = json_unify['data']
    self.data_list = data_list
    self.filename = None
    self.tableName = None
    self.query = ""
    
  def create(self, name):
    retVal = ""
    drop = "DROP TABLE IF EXISTS " + name + ";\n"
    
    self.query += drop + '\n'
    create = "CREATE TABLE "+name+" ("
    
    self.query += create + '\n'
    

  def values(self, name):
    
    queries = ""
    
    # get data as rows / array
    
    # for each row, create query
    for row_index, row in enumerate(self.data_list[1:]):
      
      query = "INSERT INTO "+name+"("+self.get_column_names()+") VALUES ("      
      vals = ""
      for index, item in enumerate(row):        
        vals += "'"+str(item)+"'"
        if index < len(row)-1:
          vals += ', '
      
      query += vals + ');'
      
      self.query += query + '\n'
    # add to queries
    
  def set_schema(self, schema):
    keys = list(schema.keys())
    
    for key in keys:
      
      row = '\t'+key + ' ' + schema[key] +'\n'
      self.query += row

  def get(self, params):
    
    name = params['table']  
    self.tableName = name
    if 'file' in params:
      self.filename = params['file']
    
    retVal = ""
    self.create(name)
    

    if 'schema' in params:
      self.set_schema(params['schema'])
    else:
      self.get_column_names_and_types()
    end_parenthesis = ');\n'
    
    self.query += end_parenthesis + '\n'
    self.values(name)
    self.add_column_comments()
    print(self.query)
    if self.filename != None:
      with open(self.filename, "w") as outfile:
        outfile.write(self.query) 
    #return retVal

  def add_column_comments(self):
    self.query += '\n'
    headers = self.json_unify['concepts']['headers']
    desc = headers['description']
    for index, header in enumerate(headers['name']):    
      
      query = 'COMMENT ON COLUMN '+str(self.tableName)+'."'+header+'" IS \''+str(desc[index])+'\';\n'
      self.query += query
    
    


  def get_column_names(self):
    keys = list(self.data.keys())
    rows = ""
    for index, key in enumerate(keys):
      if index < len(keys)-1:
        line = '"'+key+'", '
      else:
        line = '"'+key+'"'+' '
      rows += line
    return rows

  def get_types(self):
    types = self.json_unify['concepts']['headers']['type']
    
    retVal = []
    for t in types:
      if t == "string" or t == "text":
        retVal.append('text')
      elif t == None or t == 'None':
        retVal.append('text')
      elif t == "bool":
        retVal.append('boolean')
      elif t == "float":
        retVal.append('double precision')
      elif t == "int":
        retVal.append('bigint')
      elif t == "json":
        retVal.append('jsonb')
    return retVal
      

  def get_column_names_and_types(self):
    keys = list(self.data.keys())
    types = self.get_types()
    
    rows = ""
    for index, key in enumerate(keys):
      if index < len(keys)-1:
        data_type = types[index]+','
      else:
        data_type = types[index]
      key_name = '"'+key+'"'+' '
      line = "\t"+key_name+' '+data_type+" "
      rows += line
      
      self.query += line + '\n'
      
```
