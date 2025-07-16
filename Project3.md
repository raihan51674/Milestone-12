### Image Upload use ImgBB :
```js
1.get api key form the Imgbb :
2.FRONTEND:
 <label htmlFor="name" className="block text-sm font-medium text-gray-700 mb-1">
              Upload profile picture
            </label>
            <input
              id="name"
              type="file"
              onChange={handleUploadImg}
              placeholder="Your Name"
              className="w-full px-4 py-3 rounded-xl border border-gray-300 focus:outline-none focus:ring-2 focus:ring-purple-400"
            />

   const [profilePic, setProfilePic] = useState('');
   const handleUploadImg = async (e) => {
    const image = e.target.files[0];
    console.log(image)

    const formData = new FormData();
    formData.append('image', image);


    const imagUploadUrl = `https://api.imgbb.com/1/upload?key=36f47c5ee620bb292f8a6a4a24adb091`
    const res = await axios.post(imagUploadUrl, formData)

    setProfilePic(res.data.data.url);
  }

and update firebase :

// add AuthProvider
 //const UpdateProfile= (profileInfo)=>{
   // return updateProfile(auth.currentUser, profileInfo)
 // }
  createUser(data.email, data.password)
      .then((result) => {
        console.log(result.user);
        // update user profile in firebase
        const userProfile = {
          displayName: data.name,
          photoURL: profilePic
        }
        UpdateProfile(userProfile)
          .then(() => {
            console.log('profile name pic updated')
          })
          .catch(error => {
            console.log(error)
          })
      })


```
