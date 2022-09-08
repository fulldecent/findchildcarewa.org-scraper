# findchildcarewa.org scraper

This project assists you to access the public data at findchildcarewa.org and save to a usable CSV file.

## Why?

The findchildcarewa.org is the official website listing child care providers in Washington. The information is public and in response to a Washington right to know law request were a reference to the website.

## Step 1 get keys

The website provides access keys for each data element.

```sh
curl 'https://www.findchildcarewa.org/apexremote' \
  -H 'Content-Type: application/json' \
  -H 'Referer: https://www.findchildcarewa.org/PSS_Search?p=DEL%20Licensed' \
  --data-raw '{"action":"PSS_SearchController","method":"getSOSLKeys","data":["","",["DEL Licensed"],[],null,null,null,[]],"type":"rpc","tid":2,"ctx":{"csrf":"1","vid":"","ns":"","ver":1}}' |
  jq '.[0].result' > keys.json
```

## Step 2 get result pages

Install node packages axios using Yarn.

Then run this script.

```javascript
import fs from 'fs';
import axios from 'axios';
const keysFile = fs.readFileSync('./keys.json');
const keys = JSON.parse(keysFile);

for (let i = 0; i < keys.length; i+=10) {
    const keysSlice = keys.slice(i, i+10);
    console.log(i);

    // Make synchronous HTTP request to get data
    const response = await axios.post('https://www.findchildcarewa.org/apexremote', {
        "action": "PSS_SearchController",
        "method": "queryProviders",
        "data": [keysSlice],
        "type": "rpc",
        "tid": 5,
        "ctx": {
            "csrf": "1",
            "vid": "",
            "ns": "",
            "ver": 1
        }
    }, {
        headers: {
            'Content-Type': 'application/json',
            'Referer': 'https://www.findchildcarewa.org/PSS_Search?p=DEL%20Licensed'
        }
    });

    // Save data to file
    fs.writeFileSync(`pages/${i}.json`, JSON.stringify(response.data));
}
```

## Step 3 combine results

```sh
cat pages/*json | jq '.[0].result' | jq -s add > all.json
```

## Step 3 convert to CSV

Install json2csv NPM CLI package. And then:

```sh
json2csv < all.json > all.csv
```

