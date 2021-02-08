# fio-search

A minimal web UI to handle searching if a [FIO](https://fioprotocol.io) domain or address (name@domain) has
been registered. Not much more to it than that. There's a live example on my site [blockpane.com](https://snap.blockpane.com/fio-search/index.html)
available.

Performing a table query for FIO addresses is a little tricky, so although simple, this tool provides 
a decent in-browser example of building the query (see App.vue for 
a full example):

```vue
<script>
import axios from 'axios';
export default {
  name: 'App',
  data () {
    return {
      input: "",
    }
  },
  methods: {  
    getAddress() {
        // 'this.input' is the address being searched:
        // convert it by a byte array
        const encoded = new TextEncoder().encode(this.input)
  
        // get a sha-1 hash of the value
        crypto.subtle.digest("SHA-1", encoded).then( hash => {
  
          // take the first 16 bytes of the hash, and convert to big-endian
          const hashArray = Array.from(new Uint8Array(hash)).slice(0,16).reverse()
  
          // convert to a string with '0x' prefix
          const hashHex = '0x' + hashArray.map(b => b.toString(16).padStart(2, '0')).join('')
  
          // send off the query:
          axios.post("https://fio.blockpane.com/v1/chain/get_table_rows", {
            code: "fio.address",
            scope: "fio.address",
            table: "fionames",
            // hashHex is the converted value
            lower_bound: hashHex,
            upper_bound: hashHex,
            limit: 1,
            // note the key_type and index_positions
            key_type: "i128",
            index_position: "5",
            json: true,
            reverse: true
          }).then(res=> {
            if (res.data.rows != null && res.data.rows[0] != null) {
              // handle the result here....
              console.log(res.data.rows[0])
          }).catch(error=> {
            console.log(error)
          })
      })
    },
  }
}
</script>
```

## Project setup
```
npm install
```

### Compiles and hot-reloads for development
```
npm run serve
```

### Compiles and minifies for production
```
npm run build
```

### Lints and fixes files
```
npm run lint
```

### Customize configuration
See [Configuration Reference](https://cli.vuejs.org/config/).
