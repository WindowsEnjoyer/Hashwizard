# hashwizard

is a suite of tools and APIs that allow you to interact with `hashing`

## Get

### Docker

The application is dockerized to simulate app on low memory enviorments

```shell
docker build -t hashwizard-dev .
docker run -p 8080:3000 --memory=512m hashwizard-dev # forward 3000 to 8080
```

### Server

```shell
node ./index.js
```

```shell
--port=3000 # explicitly specify the port
--no-dump # disables fetching initial load
```

**NOTE**: if `--no-dump` option is set, by default the app will return an `internal-error`, please refer to [`./routes/crack.js`](./routes/crack.js#L57). this measure is put into place since our app runs in limited memory of `512 MB` which is the free tier for [render.com](render.com)

### Test

```shell
node ./tests/*.test.js
```

### Miscellaneous

```shell
node ./lib/DumpRemoteTableToFile.js 
```

This attempts to fetch a large dataset from mongoDB via [`./models/HashModel.js`](./models/HashModel.js) and break it down into chunks which are categorized by algorithm.

```shell
# output generated by "node ./lib/DumpRemoteTableToFile.js "
 
./data
├── data-md5
├── data-sha1
└── data-sha256
```

```shell
node ./lib/UpdateDBFromLocalIndex.js
```

This takes a `.txt` file as the only argument. This text file is uploaded AS-IS to the database and is later fetched-from through the `DumpRemoteTableToFile` 

## Notes on Memory optimization

Several measures have been implemented to allow us to deal with a data base of more than 100 MB, here are a few of them
- Chunking: when the large text dataset is pulled from the database in `DumpRemoteTableToFile` it is broken down into chunks of 1500

## API

### `/api/hash/`

hash the provided text using the `{hash}` function



- `/:hash`: the hash function to use; supports md5, sha256, sha1, sha224, sha256, sha384, sha512, sha3-224, sha3-256, sha3-384, sha3-512, shake128, shake256

#### Query params

- `text=`: string of characters to hash

#### Success

```json
{
   "hash" : "5d41402abc4b2a76b9719d911017c592",
   "status" : 200,
   "text" : "hello"
}
```

#### Error

```json
{
   "error" : "invalid-hash-function",
   "message" : "the hash function specified is not valid",
   "status" : 400
},

{
   "error" : "missing-text-param",
   "message" : "the text parameter was not provided for `/api/hash`",
   "status" : 400
}
```

### `/api/crack/`

Attempts to crack a given hash via a custom 20+ MB [Rainbow table](https://en.wikipedia.org/wiki/Rainbow_table)

- `/:hash`: the hash function to use; supports md5, sha256 and sha1

### Query params

- `hash=`: Hash to be cracked

#### Success

```json
{
   "hash" : "5d41402abc4b2a76b9719d911017c592",
   "status" : 200,
   "text" : "hello"
},

{
   "hash" : "e",
   "message" : "was unable to crack the given hash",
   "status" : 200,
   "text" : null
}
```

#### Error

```json
{
   "error" : "missing-hash-param",
   "message" : "the hash parameter was not provided for `/api/crack`",
   "status" : 400
},

{
   "error" : "invalid-hash-function",
   "message" : "the hash function specified is not valid",
   "status" : 400
}
```

### Credits

- [danielmiessler/SecLists](https://github.com/danielmiessler/SecLists/) for the data provided for the Rainbow table
