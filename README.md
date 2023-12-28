# Verify Ton Rocket Fair
This game uses two ingredients:

*Server Seed* - provided by us
*Client Seed* - the hash from Ethereum block

We take a Server Seed and hash it (SHA256), creating a new Server Seed. Then we take that Server Seed and hash that too. We repeat this process until we have 4 million hashes -- 4 million server seeds. The very first game uses the 4 millionth server seed (in the codesandbox below), and each game after that works backwards down the list of server seeds. Second game uses the 3,999,999th hash and so on and so forth.

The Client Seed is a public key that we agree to using before we know what it is. We commit to the hash of a crypto block before that block is mined, and we create all the server seed hashes before committing to this block. This ensures that we could not control the outcome of each game. The current Client Seed is in the codesandbox below.

To verify this, you can input the Server Seed to your game in the code below, and you should see the server seeds and game results for the previous 100 games.

```
const crypto = require("crypto");


// Write Last Game Hash Here
const crashHash = "";

// Client Seed: hash of ETH block 18841250
const tonRocketClientSeed =
  "0xc582b3fb66319ab47ab592291c9799a22ff21c5d40733e159a04d663dec2ce6e";

function generateHash(seed) {
  return crypto.createHash("sha256").update(seed).digest("hex");
}

function divisible(hash, mod) {
  // We will read in 4 hex at a time, but the first chunk might be a bit smaller
  // So ABCDEFGHIJ should be chunked like  AB CDEF GHIJ
  var val = 0;

  var o = hash.length % 4;
  for (var i = o > 0 ? o - 4 : 0; i < hash.length; i += 4) {
    val = ((val << 16) + parseInt(hash.substring(i, i + 4), 16)) % mod;
  }

  return val === 0;
}

function crashPointFromHash(serverSeed) {
  const hash = crypto
    .createHmac("sha256", serverSeed)
    .update(tonRocketClientSeed)
    .digest("hex");

  const hs = parseInt(100 / 4);
  if (divisible(hash, hs)) {
    return 1;
  }

  const h = parseInt(hash.slice(0, 52 / 4), 16);
  const e = Math.pow(2, 52);

  return Math.floor((100 * e - h) / (e - h)) / 100.0;
}

function getPreviousGames() {
  const previousGames = [];
  let gameHash = generateHash(crashHash);

  for (let i = 0; i < 100; i++) {
    const gameResult = crashPointFromHash(gameHash);
    previousGames.push({ gameHash, gameResult });
    gameHash = generateHash(gameHash);
  }

  return previousGames;
}

function verifyCrash() {
  const gameResult = crashPointFromHash(crashHash);
  const previousHundredGames = getPreviousGames();

  return { gameResult, previousHundredGames };
}

console.log(verifyCrash());
```
