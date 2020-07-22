# fb-video-upload

`fb-video-upload` allows you to easily upload videos, large or small, using the Facebook Graph API.

Works with: 
  - Pages
  - Ad Accounts
  - Personal Profile

# New Release!

  - Facebook Graph API v7.0
  - ~~Facebook Graph API v6.0~~


### Installation

`fb-video-upload` requires [Node.js](https://nodejs.org/) v4+ to run.

Install `fb-video-upload`:

```sh
$ npm i @justenbphelps/fb-video-upload
```

### Development

Include package:
```js
const upload = require('fb-video-upload')
```

Usage:
```js
const fs = require('fs');
const fbUpload = require('fb-video-upload');

const args = {
  token: "YOURTOKEN", // with upload permissions
  id: "YOURID", // {page_id || user_id || event_id || group_id}
  stream: fs.createReadStream('./fixture.mp4'), // stream & path to the video,
  title: "video title",
  description: "video escription",
  thumb: {
    value: fs.createReadStream('./fixture.jpg'),
    options: {
      filename: 'fixture.jpg',
      contentType: 'image/jpg'
    }
  }
};

fbUpload(args).then((res) => {
    console.log('res: ', res)
}).catch((err) => {
    console.error(err)
})
```

### Todos
 - Better documentation!
 - Clear up syntax for readability

## License
MIT © [justenbphelps](https://github.com/justenbphelps)