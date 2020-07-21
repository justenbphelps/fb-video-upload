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
$ npm i @justenbphelps/fb-video-upload@1.0.0
```

### Development

Include package:
```js
const upload = require('fb-video-upload')
```
Usage:
```js
const streamToPromise = require('stream-to-promise')
const rp = require('request-promise')

const url = 'https://graph-video.facebook.com'
const version = 'v7.0'

const retryMax = 10
let retry = 0

const apiInit = (args, videoSize) => {
  const options = {
    method: 'POST',
    uri: `${url}/${version}/${args.id}/advideos?access_token=${args.token}`,
    json: true,
    form: {
      upload_phase: 'start',
      file_size: videoSize
    }
  }

  return rp(options).then(res => res)
}

const apiFinish(args, upload_session_id, video_id) => {
  const {token, id, stream, ...extraParams} = args

  const options = {
    method: 'POST',
    json: true,
    uri: `${url}/${version}/${args.id}/advideos`,
    formData: {
      ...extraParams,
      upload_session_id,
      access_token: args.token,
      upload_phase: 'finish',
    }
  }

  return rp(options)
    .then(res => ({...res, video_id}))
}

const uploadChunk = (args, id, start, chunk) => {
  const formData = {
    access_token: args.token,
    upload_phase: 'transfer',
    start_offset: start,
    upload_session_id: id,
    video_file_chunk: {
      value: chunk,
      options: {
        filename: 'chunk'
      }
    }
  }
  const options = {
    method: 'POST',
    uri: `${url}/${version}/${args.id}/advideos`,
    formData: formData,
    json: true
  }

  return rp(options)
    .then(res => {
      retry = 0
      return res
    })
    .catch(err => {
      if (retry++ >= retryMax) {
        return err
      }
      return uploadChunk(args, id, start, chunk)
    })
}

const uploadChain = (buffer, args, res, ids) => {
  if (res.start_offset === res.end_offset) {
    return ids
  }

  var chunk = buffer.slice(res.start_offset, res.end_offset)
  return uploadChunk(args, ids[0], res.start_offset, chunk)
    .then(res => uploadChain(buffer, args, res, ids))
}

const facebookApiVideoUpload = (args) => {
  return streamToPromise(args.stream)
    .then(buffer => Promise.all([buffer, apiInit(args, buffer.length)]))
    .then(([buffer, res]) => uploadChain(buffer, args, res, [res.upload_session_id, res.video_id]))
    .then(([id, video_id]) => apiFinish(args, id, video_id))
}

module.exports = facebookApiVideoUpload

```

### Todos

 - Better documentation!
 - Clear up syntax for readability

## License
MIT © [justenbphelps](https://github.com/justenbphelps)