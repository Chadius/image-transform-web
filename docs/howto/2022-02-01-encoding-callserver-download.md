# Spike results
I'm blundering into my first foray into Front End programming. This will serve as a sample of my efforts.

# Lessons learned
## Making the request
- Import the TwirpScript client so you can adjust the URL using `client.baseURL`
- Import the `service.pb.ts` file so you can access the proto methods.
- The methods are asynchronous, so feel free to `await` them.

```typescript
import { client } from "twirpscript";
import { Transform } from '../../protos/service.pb';

client.baseURL = "http://localhost:8080";
const imageResponse = await Transform({
  inputImage: inputImage,
  formulaData: formulaData,
  outputSettings: outputSettings
});
```

`imageResponse.imageData` has the actual data.

## Encoding data into a byte array
Twirp uses `Uint8Array` for data, so you'll have to convert data streams into it.

There are many, many ways to encode a data file. It's hard to paste the file as a stream of data (I tried `cat filename | pbcopy` but JS didn't like my pasted data.)
So I'm using a base64 encoded file, and `atob` to convert it to a byte array. `atob` works on ASCII and converts to bytes (base64 is in ASCII.)
Once you have the byte array:

- Use `charCodeAt` to convert each character to its ASCII byte code.
- Create a `Uint8Array` with the numerical byte array. 

```typescript
let imageBase64String = "iVBORw0KGgoAAAANSUhEUgAAAMYAAADGCAYAAACJm/9dAAAAAXNSR0IArs4c6QAAAkVJREFUeJzt3LFtQzEQBUF+gbW4G6sKN6ZIqQG1ptAAHdvYnAxmKrhk8bK71lprAH/cdh8AJxIGBGFAEAYEYUAQBgRhQBAGBGFAEAYEYUAQBgRhQBAGBGFAEAYEYUAQBgRhQBAGBGFAmO/77hPgPBYDgjAgCAOCMCAIA4IwIAgDgjAgCAOCMCAIA4IwIAgDgjAgCAOCMCAIA4IwIAgDgjAgCAPCfO6+AA5kMSAIA4IwIAgDgjAgCAOCMCAIA4IwIAgDgjAgCAOCMCAIA4IwIAgDgjAgCAOCMCAIA4IwIMzXx+4T4DwWA4IwIAgDgjAgCAOCMCAIA4IwIAgDgjAgCAOCMCAIA4IwIAgDgjAgCAOCMCAIA4IwIAgDwvW1PtfuI+A0FgOCMCAIA4IwIAgDgjAgCAOCMCAIA4IwIAgDgjAgCAOCMCAIA4IwIAgDgjAgCAOCMCAIA8K1fnwJgf8sBgRhQBAGBGFAEAYEYUAQBgRhQBAGBGFAEAYEYUAQBgRhQBAGBGFAEAYEYUAQBgRhQBAGhDm+d58A57EYEIQBQRgQhAFBGBCEAUEYEIQBQRgQhAFBGBCEAUEYEIQBQRgQhAFBGBCEAUEYEIQBQRgQ5njsPgHOYzEgCAOCMCAIA4IwIAgDgjAgCAOCMCAIA4IwIAgDgjAgCAOCMCAIA4IwIAgDgjAgCAOCMCBcY4y1+wg4jcWAIAwIwoAgDAjCgCAMCMKAIAwIwoAgDAjCgCAMCMKAIAwIwoAgDAjCgCAMCMKAIAwIwoDwC4bFEJeYLAMpAAAAAElFTkSuQmCC";
const byteCharacters = atob(imageBase64String);
const byteNumbers = new Array(byteCharacters.length);
for (let i = 0; i < byteCharacters.length; i++) {
  byteNumbers[i] = byteCharacters.charCodeAt(i);
}
const inputImage = new Uint8Array(byteNumbers);
```

### Encoding text into Uint8Array
You can use `TextEncoder` to do this. It assumes your text is UTF-8.
I've added faking JSON here, but I could probably use JSON.Stringify here.

```typescript
var enc = new TextEncoder(); // always utf-8
const formulaData = enc.encode(
  `INSERT YAML/JSON HERE`
);

const outputSettings = enc.encode(
`{'output_width':200,'output_height':200}`
);
```

## Forcing a download
I can't initiate a download automatically. Technically the user needs to click on it.
- So in `downloadURL` here, JS creates a link that points to the resource. 
- Then it clicks on the link to start the download before removing it. 

```typescript
const downloadURL = (data, fileName) => {
  const a = document.createElement('a')
  a.href = data
  a.download = fileName
  document.body.appendChild(a)
  a.style.display = 'none'
  a.click()
  a.remove()
}
```

You can't download without a URL and a MIME type.
So given some data, filename and its type, you'll need to:
- Make a blob of data with the given MIME type, so it knows what to download.
- Create a URL for the blob
- Force a download using the `downloadURL` function
- Wait for the download to start before revoking the URL (Or you won't be able to download it immediately)

```typescript
const downloadBlob = (data, fileName, mimeType) => {
  const blob = new Blob([data], {
    type: mimeType
  })
  const url = window.URL.createObjectURL(blob)
  downloadURL(url, fileName)
  setTimeout(() => window.URL.revokeObjectURL(url), 1000)
}
```

### Actually calling the download
Once you have the data and the user wants to download, go ahead and initiate.

```typescript
downloadBlob(imageResponse.imageData, "response.png", "image/png");
```