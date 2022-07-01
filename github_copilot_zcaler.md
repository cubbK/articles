# Make Github Copilot Work in Wsl With Zscaler

TLDR: replace every occurence of `rejectUnauthorized: x` to `rejectUnauthorized: false` in `~/.vscode-server/extensions/github.copilot-xxx/dist/extension.js` and restart vscode

There is an ongoing issue with copilot in vscode in that it doesn't pick up the system chain of certificates and instead uses a [hardcoded list](https://github.com/nodejs/node/issues/4175) of certificates. This causes problems with zscaler and similar man in the middle apps that causes it to fail to activate because it doesn't trust the certificate and gives the error `Extension activation failed: "unable to get local issuer certificate"`.

There are several ways to fix this proposed on the internet:

- use [win-ca](https://github.com/ukoloff/win-ca) or [mac-ca](https://github.com/jfromaniello/mac-ca) vscode extension to make available the additional certificates
- add zscaler self-signed certificate in Chrome by going to `chrome://settings/privacy` [more](https://stackoverflow.com/a/60476625)
- run `code --ignore-certificate-errors`

The only way that worked though and unfortunately the most hacky is to change in the extension bundle `rejectUnauthorized` to `false`

- open `~/.vscode-server/extensions/github.copilot-xxx/dist/extension.js`
- search every occurrence of `rejectUnauthorized:[a-z]` regex find and change to `rejectUnauthorized: false`
- Important!! In some cases `rejectUnauthorized` will be as part of an destructured object. For example when having

```js
const {
  h1: r,
  options: { h1: i, rejectUnauthorized: s },
} = e;
```

change it to

```js
const {
  h1: r,
  options: { h1: i, rejectUnauthorized: s },
} = { ...e, options: { ...e.options, rejectUnauthorized: false } };
```

- restart vscode

Based on https://github.com/github-community/community/discussions/8866 and https://stackoverflow.com/questions/36506539/how-do-i-get-visual-studio-code-to-trust-our-self-signed-proxy-certificate
