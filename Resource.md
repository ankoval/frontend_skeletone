### DRAFT --  DRAFT --  DRAFT --  DRAFT --  DRAFT

## The goal
When we work with REST resources, in 90% of cases, all we need is to get data and render or bind to the form and save after the change.
This rather nontrivial task leads to a large amount of the same type of code. Sort of:
```
// action types
const FETCH_USERS_LIST = Symbol('FETCH_USERS_LIST')
const FETCH_USER = Symbol('FETCH_USER')
const SAVE_USER = Symbol('SAVE_USER')
const CREATE_USER = Symbol('CREATE_USER')

// action creators
function fetchUsersList() {
  return {
    type: FETCH_USERS_LIST
  }
}

function fetchUser(id) {
  return {
    type: FETCH_USER,
    payload: id,
  }
}

function saveUser(id, data) {
  return {
    type: SAVE_USER,
    payload: data,
    meta: {id}
  }
}

function createUser(data) {
  return {
    type: CREATE_USER,
    payload: data,
  }
}

// reducers
// TODO
function users(state = {}, action) {
  switch(action.type) {
    case FETCH_USERS_LIST:
      return {...state, isLoading: true}
  }
}

// epics
// TODO
...
```
And this example does not contain error handling, filtering, pagination, caching, JWT, options, etc.
At the same time, such code is repeated again and again in each store file and essentially differs only in the name of the functions and constants:
```
fetchUsers
fetchBooks
fetchGroups
fetchComputers
fetchOrders
etc
```
resource.js gives you the opportunity to connect each API resources without much difficulty.
```
connectResource(resource, options)

resource = {
 namespace: 'internalResourceName',
 endpoint: '/some/endpoint/with/:placeholders'
}
options = { /* others */ }
```
then `props` got structure:
```
props.internalResourceName = {
  data: { /* ... resource data from API ... */ },
  isLoading: false, // or true,
  options: { /* parsed OPTIONS from API */ },
  errors: null, // or object with errors { },
  loading: 0, // number of currently loadings, used internaly
  
  // actions
  fetch: func, // GET request, useful when no prefetch
  create: func, // POST request
  save: func, // PATCH request
  update: func, // PATCH request, alias for save
  remove: func, // DELETE request
  replace: func, // PUT request if you need it
  fetchOptions: func, // OPTIONS request
  setData: func, // you can update data in store manually, but please be carefull with this action
  setErrors: func, // you can updates errors in store manually, but please be carefull with this action
  setFilters: func, // you can updates current filters in store manually, but please be carefull with this action
  
  filters: {}, // current applied filters
  filter: func, // action to re-fetch resource with new filters

  // filtering (using browser query) TODO
  withRouter: false, // or true
  navigate: func, // the same as filter({offset: newOffset})
}
```


## Options

almost all options can be set both at the resource configuration level and at the connection level

#### `namespace : String|Array` [required]

property name for resource binding. e.g. for `namespace: 'test'` you will get resource in `props.test`
you can set it as array with two strings. useful for list resource. e.g for `namespace: ['books', 'book']` you will get `this.props.books` for list request, or `this.props.book` for single item request (see `list` option for details)

#### `endpoint : String` [optional] [default: value of namespace option]

will be set to `namespace` if ommited. resource endpoint name - where to get data from server.
can contains placeholders (see doc for `path-to-regexp` package for additional information). all you current props will be forwarded to `path-to-regexp`, plus one additional property `id` wich will be get by `idKey` (see below)

#### `list : Boolean` [optional] [default: false]

mark you resource as list resource. you endpoint should conins `:id?` placeholder, e.g. `accounts/:id?`
when you have id property (this props name can be changed via `idKey`) in props then the item resource will be binded. otherwice list. 

#### `idKey : String` [optional] [default: id]

name for the property that contains id for item. see `list` option for details

#### `prefetch : Boolean` [optional] [default: true]

should the resource be prefetched. if you set this to true you will get `props[namespace].data === null`. useful for custom resources that are not previously created on the server.

#### `refresh : Boolean` [optional] [default: false]

should the resource be refreshed when you render this component and the data for this resource already exists in store

#### `form : String|undefined` [optional] [default: undefined]

this property doesn't make sense when ommited. in `connectFormResource` this field is required
connect resource for redux-form. you should specify form name here for properly validation working. 
this option means that you will also get `initialValues` and `onSubmit` props. 
`initialValues` will be set to prefetched data from server, or empty object (when no `prefetch === false`)
`onSubmit` will do `create` (POST) for new items (`prefetch === false` means that item is new, for list resources missed id also means that the resource item is new). for existed items `onSubmit` will do `update` (PATCH)

#### `item : Boolean` [optional] [default: Boolean(form)]

used only for list resources, only inside options. will be set to true when form name is specified. this property means that we working with item inside list. sometimes you need to make "create item form". 

#### `options : Boolean` [optional] [default: false]

prefetch OPTIONS from endpoint (useful for geting choices for selects from API)

#### `async : Boolean` [optional] [default: false]

should the component be rendered without data

#### `filters : Object` [optional] [default: {}]

used with list resources. representing initial query for fetch request.


### Examples

```
compose(
  // ypu can omit this for create boook form. when id exists that ths is edit book form
  connect(_ => ({uuid: 'e4831316-9e2a-41b7-bb77-514318ac51ba'})),
  connectFormResource({
    namespace: ['books', 'book'],
    endpoint: 'books/:id?',
    list: true,
    options: true,
  }, {form: 'book'}),
  reduxForm({
    form: 'test',
  })

  // you can mix resources inside one component
  connectSingleResource({
    namespace: 'config', // ommited endpoint will be set to 'config'
  }),
)
```
