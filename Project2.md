### 1.Add axiosSecure :
```js
import React from 'react';
import axios from 'axios'

const useAxiosSesure = () => {

  const instance = axios.create({
  baseURL: `http://localhost:3000`
});
  return instance
    
  }
export default useAxiosSesure;



//use
  axiosSecure.post("/parcels",ParcelData)
    .then(res=>{
      console.log(res.data);
    })

```
### 2.Tanstack Quary :
```js
1.npm i @tanstack/react-query
2.main.jsx add :
import {
  QueryClient,
  QueryClientProvider,
} from '@tanstack/react-query';
const queryClient = new QueryClient()
<QueryClientProvider client={queryClient}>
      //AuthProveider
    </QueryClientProvider>
```
### 3. TanStack Use :
```js
```
