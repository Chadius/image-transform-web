# What is TwirpScript?
https://github.com/tatethurston/TwirpScript

TwirpScript will work for JS and TS.

Make a TS script that works in a browser & communicates with the twirp server you made in go.

# Why should I use it?
- Use the Twirp RPC system
- Make calls between different languages

# Install twirpscript and prerequisites
```bash
npm install --save-dev typescript
npm install --save-dev tsc
npm install --save-dev twirpscript
```

You also need to install `yarn`.

# How to use it
1. Define your service in a `.proto` file, or copy it from your server project.
2. Run `yarn twirpscript` (yes it must be yarn) to generate JavaScript or TypeScript code from your `.proto` file. This will generate JSON and Protobuf clients, a service interface, and service utilities.
   - If you see `protos/service.pb.js` instead of `.ts` you don't have a `tsconfig.json` file.
3. Use the generated client to make requests to your server.

#### 3. Use the client

Use the generated clients to make `json` or `protobuf` requests to your server:

Browser client's `src/client.ts`:

```js

import { client } from "twirpscript";
import { MakeHat } from "../protos/haberdasher.pb";

client.baseURL = "http://localhost:8080";

const hat = await MakeHat({ inches: 12 });
console.log(hat);
```
