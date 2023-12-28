# Verify Ton Rocket Fair

This game ensures fairness through a dynamic process involving a **Server Seed** and a **Client Seed**. The Server Seed, initially provided by us, undergoes multiple hashing iterations, generating a sequence of 4 million server seeds. Each game uses a server seed from this sequence, creating a transparent and verifiable gaming environment.

The Client Seed, a public key agreed upon before revelation, is based on the hash of an Ethereum block. This commitment to the hash occurs before the block is mined, ensuring that outcomes are beyond our control. The current Client Seed is presented below.

To verify fairness, input the Server Seed into the provided code. You'll witness the server seeds and game results for the last 100 games.

Javascript
```
const crypto = require("crypto");

// Write Last Game Hash Here
const crashHash = "";

// Client Seed - hash of ETH block 18841250
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
