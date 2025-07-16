### User data store database :
```js
// create axiosInstacen jekhane thakbe jwt
//1. Registation page:
  createUser(data.email, data.password)
            .then(async (result) => {
                console.log(result.user);

                // update userinfo in the database
                const userInfo = {
                    email: data.email,
                    role: 'user', // default role
                    created_at: new Date().toISOString(),
                    last_log_in: new Date().toISOString()
                }

                const userRes = await axiosInstance.post('/users', userInfo);
                console.log(userRes.data);

//2. similar data add GoogleLogin
```
### User Api :
```js
        app.post('/users', async (req, res) => {
            const email = req.body.email;
            const userExists = await usersCollection.findOne({ email })
            if (userExists) {
                // update last log in
                return res.status(200).send({ message: 'User already exists', inserted: false });
            }
            const user = req.body;
            const result = await usersCollection.insertOne(user);
            res.send(result);
        })
```
