
## Discipl Core API

This API is a javascript Es6 module intended for client side usage. It is intended to run in NodeJS, React Native (through expo.io) and JS engines in webbrowsers. With it you can let actors create self sovereign id's, let them make claims about themselves, attest claims of others, verify claim attestations and possibly revoke claims. The self sovereign id is a DID (https://github.com/WebOfTrustInfo/rebooting-the-web-of-trust-fall2017/blob/master/draft-documents/did-primer.md) denoting a supported platform which is any platform that supports storing claims in relation to a public id, contained in the DID, and enforces that only a holder of a corresponding private key can make claims in relation to that id. The claims in relation to a particular DID form a channel and can link to claims in other channels. These claims are attestations. The API contains a function to see whether a particular claim is attested by one or more particular other DID's. Further it contains functions to retrieve claims in channels, monitor channels and even traverse or fetch all data in the linked channels. 

In essence the API is therefore a linked data client that can crawl over all used and supported platforms that attestations link to. Doing so it can export or you can traverse a linked data set for all kinds of processing like feeding a dashboard based on R with discipl-R. There are a lot of platforms that can be used: distributed public ledgers, permissioned ledgers but also more private self sovereign identity/ verifiable claim or credential solutions. The specifics of the platform used determins how safe it is to put what kind of information in claims. So note that discipl core does not prohibit you to put sensitive information in public if you choose the wrong platform. Note that self sovereign identities are meant to be relatively short lived and it's usage bounded by a use case.

Discipl Core is meant to be lean and simple and it does not impose usage of metadata on data. It only enforces and therefore structures linked data to have always self sovereign id's as subject of the linked data triples and that the holder of a private key related to this self sovereign as public key is the only one that can store such triples, we call claims. It therefore wil not neccasarily comply with standards like the verifiable credentials specification but will follow it's own path as it's usage is more broad than only verifiable credentials.

## Terminology

term | description
--- | ---
*self sovereign id* | a random identifier an actor chooses/generates for him/herself to refer to him/herself in subsequent communication. This identifier mostly is a public key belonging to a private key the holder of that key can use to make claims exclusively in relation to this self sovereign id. Within Discipl Core we use a DID like syntax for this identifier so when referring to the "DID" of an actor we also mean this self sovereign id of the actor. Note that self sovereign identities are meant to be relatively short lived and bounded by use case.
*claim* | one or more subject-predicate-object triples, typically stored as JSON-LD with no metadata where the subject is always the DID (or reference to this in the form of "I") of the actor that created the claim. The predicate is any string and required. The object, a string, is optional.
*channel* | set of claims with the same object (same DID)
*attestation* | a claim with a link to some other claim (could be a claim with the same subject, thus in the same channel). 
*link* | The link is either a platform specific reference (if the platforms supports that and it's immutabillity is to be trusted, or it is intended to be mutable), or it can contain a proof, for instance a peppered hash (HMAC-386) of (multiple) predicate-object parts of the claim with the object (a DID) as key. This DID is to be kept off ledger then by the way. So to be able to verify such proofs, you'll need to know at forehand the whole claim being attested and thus know the actual link (or you'll probably have to do a lot of work), but the proof does define a link to some otherwise unreferencable piece of information. In most cases you would get the claim being referenced from the actor that made the claim. Do not mistake a link by proof with a verifiable claim/credential for sensitive data though, on itself, it is not intended nor safe for this usage as for instance peppered hashes are not considered good enough as an anonymisation technique. The proof is intended to verify a link to a given piece of information in the underlying platform this information is stored in. The proof in these links is platform specific and may not be suitable to be stored on ledgers.<br><br>A reference can be a specific reference to a logged transaction the claim was stored with, or it can be the DID, referencing the first claim in the corresponding channel. Platforms may have limited functionality. The link always denotes the platform and does this by using a notation similair to the DID notation. Examples:<br>- *link by reference (of a DID or transaction)* : link:discipl:iotaDJI99DSKNSNDL99WDSDNSDNLLWNWNWQQNLSDNSDN9SD999SDSDNKNJSND9DSHPLVZ<br>- *link by proof* : link:discipl:zenroomLIwew77897wer[=87s9ydh]sd999sdnkLIJEOJJ#*Y*YOjjjhskds93k4n3knk9fs<br>(above examples are just an indication)
*platform* | any operational technology capable of storing/retrieving claims in relation to self sovereign id's
*linked data export* | given a DID, one can export all claims of that DID and all claims in channels with claims linked by reference in those claims and all claims in channels with claims referenced in those claims and so on. Stopping at circulair references one can export or iteratively traverse a whole dataset. Note that a separate DID should be used for a single proces in a single use case which should make the dataset limited. Dependent on the use case it can become fairly big though.

## supported platforms

Many platforms can be supported to become part of the discipl platform. We distinguish the following platform types:

- local wallet 	: platforms that store claims in local wallets within platform specific apps or platforms that are implemented as libraries for local storage of claims
- public ledger	: permissionless public distributed ledgers. 
- hybrid 			: permissionless public distributed platforms that combine both public ledger as local wallet platform type of functionality 
- hybrid managed	: permissioned, centrally managed distributed ledgers that might be GDPR compliant enough to store sensitive data in.

At the moment just a few have initial implementations, most are planned, others desired. All these supported platforms have their own connector repository. platform connectors implement the connector interface defined here.

Platform		| Type 				| Status		| Connector Repository			| Usable for PII | duration	| requires node | Remarks
--- | --- | --- | --- | --- | --- | --- | ---
Memory			| local wallet		| implemented	| discipl-core-mem				| yes			 | <0.1s	| no, only local| you must add memory protection yourselves, just a local store of claims in memory as long as process it is used in runs
IOTA 			| public ledger		| implemented	| discipl-core-iota				| no			 | 30s		| yes, manual	| uses IOTA MAM feature in public mode and Carriota field, retaining claims after each milestone requires permanodes. MAM protected and private mode are not GDPR compliant. Note: IOTA uses a central coordinator which approves transactions but it does not do more than that; it is intended to be removed as soon as possible
IPV8			| hybrid			| planned		| discipl-core-ipv8				| yes			 | <0.1s 	| possible		| can use local svc or svc on node
Validana		| hybrid managed	| planned		| discipl-core-validana			| possible		 | 1s		| yes, manual	| if used in a permissioned way may be usable for PII if conforming to GDPR
Zenroom			| hybrid			| planned		| discipl-core-zenroom			| yes			 | <0.1s	| no, local svc	| discipl-zenroom integration; can be used to have discipl functionality within smart contract vm's that embed zenroom and is using the DECODE platform
Leopard Ledger	| hybrid managed	| planned		| discipl-core-leopardledger	| possible		 | 1s		| yes, manual	| if used in a permissioned way may be usable for PII if conforming to GDPR
IRMA			| local wallet		| planned		| discipl-core-irma				| yes			 | <0.1s	| yes, manual	| scheme of public keys and attestations is governed in a centralised way
Sovrin			| local wallet		| investigated	| discipl-core-sovrin			| yes			 |	?		| yes, manual 	| unknown 
uPort			| local wallet		| planned		| discipl-core-uport			| yes			 |	?		| yes, manual	| might require fees in future
lbtc			| public ledger		| unknown		| discipl-core-lbtc				| no			 | <0.1s	| yes, manual	| requires fees
rchain			| public ledger		| unknown		| discipl-core-rchain			| no			 |	?		| yes, manual	| requires fees
legacy			| hybrid managed	| planned		| discipl-core-legacy			| possible		 | depends	| yes, manual 	| interfaces with legacy datasources that implement REST interface

Note this library and all connectors are in early experimental development. Do not use in production.

## Basic Usage

This ES6 module will export the main interface to use discipl-core. You'll have to require connector modules and if they require a node you'll need to configure them which will make them available and used through the main interface. Alternatively you can configure the discipl core module to use a discipl node (here). By default, this library expects a local discipl node. The platforms that are made available through this node will be automaticly required and configured. 

Where needed you create a self sovereign identity for a given platform, consisting of a public-private key where the public key is contained in a correspondig DID. You are responsible for persisting these keys yourself if neccasary. If you want to use a previously created identity, you must set it. Sometimes however, you do not have to set a identity for instance if you are only going to read information from a public platform.

Note that self sovereign identities are meant to be relatively short lived and bounded by use case. The discipl-4sacan and discipl-law-reg modules are intended to be able to automaticly create ssid's for actors bounded by the actor's usage of use cases within published law and regulations and can let actors choose a connector to keep them safe and automaticly remembered when needed.

```
npm install discipl-core
npm install discipl-core-iota
```

then in Node:

```
const discipl = require('discipl-core')
const session = discipl.newSession()
discipl.newSsid(session, 'iota')
discipl.claim(session, {'need':'beer'}, function (result, err) {
	if(!err) {
		console.log("Claim registered: "+result)
	} else {
		console.log("Failed to register claim: "+err);
	}
})
```

see examples folder for more example code

## API

### `newSession`
Returns a session object that will hold current states of the channels tied to self sovereign identities that are accessed in relation to this session. One Ssid can be set in relation to a session that is used as actor when making claims in this session. You can create multiple sessions in parallel for situations multiple ssid's are makeing claims at the same time.

```
newSession()
```

### `newSsid`
Creates a new Ssid for the given connector platform consisting of a DID (containing public key) and a private key. This action will open up a channel tied to this DID on the platform. Nobody will know about this however until a first claim is published publicly (depends on platform). The new Ssid is automaticly set to be used in relation to the given session. The connector name should be given as connector argument. By default this is to be the same as the part after "discipl-core-" in the name of the connector repository in github. So for instance, for IOTA it is "iota" (lowercase).

```
newSsid(session, connector)
```

### `setSsid`
Sets a Ssid in relation to the given session. The given private key is held locally in the session object. The Ssid should be provided as a JSON object in the form of {did:pkey} You can make a session "forget" a Ssid by providing empty values: {"":""}

```
setSsid(session, ssid)
```

###	`getSsid` 
Returns the Ssid currently set in the session object. The Ssid is returned as a JSON object in the form of {did:pkey} 

```
getSsid()
```

### `claim`
Makes the Ssid set in the current session make one or more claims with the given data in it. The given data should be a JSON object with a list of key:value pairs in which the keys are predicates and the values the objects. objects can nest predicate-object pairs which may cause links to self owned claims to be stored. A connector may also store this kind of nested claims in a nested way though. Note: you will not be able to reference, attest and thus verify single nested claims. Storing a claim on a platform can take some time, dependent on the platform used. Therefor this method is asynchronous and takes a handler function that defines two (result and error) arguments as third argument. If no error occurred the handler function receives a link to the made claim as result. This link can be used in other claims (in channels of other Ssids for instance) as attestation.

```
claim(session, data, callback)
```

Example:
```
discipl.claim(session, {'need':'beer'}, function (result, err) {
	if(!err) {
		console.log("Claim registered: "+result)
	} else {
		console.log("Failed to register claim: "+err);
	}
})
```

### `link`
Creates a link to the given claim to be used in attestations. The claim can be a reference or a claim (in the form of JSON: {subject:did, predicate:object, predicate:object, ...}) and should be available in the platform used by the given connector indication (the connector name) though this is not enforced. Note, as the claim, attest, verify and revoke methods accept links in their arguments you probably don't need to use this method directly. 
```
link(connector, claim)
```

###	`attest`
Creates a simple attestation using a given predicate and claim link (as returned by the link method). In itself this is the same as making the Ssid set in the given session a claim that includes the given link. It is a shorthand for: claim(session, {predicate:link}, ...). See also the description at the claim method. To make more complex attestations, holding extra information in relation to the attestation, you can simply use the claim method with links as objects in relation to given predicates. 

```
attest(session, predicate, link, callback)
```

### `verify`
Let's you verify whether a given claim is attested by one of a given set of actors (as identified through the given ssid's) using the given predicate and returns true if so. You can also just give one ssid instead of a list. This method returns false if either the claim or all matching attestations are revoked.

```
verify(claim, predicate, ssids[])
``` 

### `get`
Returns the claim as a JSON-LD object the given link links to. The session object is optional but may be needed if the information retrieval is permissioned in which case the current Ssid set is used for identification. 

```
get(link, (session))
```

### `export`
exports linked claim data, starting at the first claim in the channel of the given DID or claim link as reference, following links to other channels which get exported in a nested dataset also and so on. When giving a DID, the whole channel is read. When giving a link to a claim, only that claim is processed. The export stops at circular references or at the given depth level. You can use this method again to expand the exported dataset even further. Just as like with the get() method, retrieving infrormation that is permissioned may require identification for which the Ssid set in a given session object may be used. You can expand using a different ssid.

The exported dataset is a JSON object containing a list of JSON objects containing an exported claim with a link as key. The claim itself, is a JSON object that can contain expanded or unexpanded links. unexpanded links contain special values indicating the reason that the link was not expanded which can be eiter:

- MAX-DEPTH-REACHED 	: the case in which the given export depth was reached 
- NOT_PERMITTED 		: the case in which the platform requires a current ssid set in a given session object to identify itself with a private key and this ssid was not set or not permitted to access the linked claim
- NOT_ACCESSIBLE		: the case in which the linked claim is not accessible, because (a proper part of) the platform is not available
- NOT_FOUND			: the case in which the linked claim could not be found (while the platform seems to be available in a sufficient way)

Dependent on the reason, further expansion of the export can be achieved. 

```
export(reference, depth, (session))
```

Example export (Note this is from an earlier version):
```
{"GVXC9TUHCULK9WLSCLGLSJZDKKVTZZOXUXGKQMCPZCUOF9CNXRJ9SLZBKYXNPHRPDJVTESS99ZSGTFAUS":{"attestedHotelier":"did:discipl:iotaURIORVZFPTWUYNGMWV9KMVVZDVWLLFLYAUNARDVWIOQHJLMDCLGWLCK9BSOLAGFKGQWLDYUPQUMV99SEN","data":{"URIORVZFPTWUYNGMWV9KMVVZDVWLLFLYAUNARDVWIOQHJLMDCLGWLCK9BSOLAGFKGQWLDYUPQUMV99SEN":{"attestedVisitor":"did:discipl:iotaYTJHGWARQITXBDLTDULVQSRKDYOHGMOBMWJOITAGIQLGPNHMBCWGIDKNONGLYSISJZZLYYGEVYVQEMWID","data":{"YTJHGWARQITXBDLTDULVQSRKDYOHGMOBMWJOITAGIQLGPNHMBCWGIDKNONGLYSISJZZLYYGEVYVQEMWID":{"attestedStay":"BZDAXEAZPFXCKTARAI9SBOEQP9M9TLXBUXEUPYUWNHZUNFIERKIKXPXLIOPNZGVYDMNBWCSBHYKCIJYHK"},"OSIFOBQGPXISIVLEPFRGSQNCPFGQFDNJZQROCDIDMFFJTUIINXHHTLJGHVOYFPDVVZQLCJCDWJZDMQMBY":{"vote":"VECOVOMYLGMINWVI9BIWSFXFDNA9LZ9CHLJOG9ODAYIHMFTZXVOKLQMMOEJZNJUIWEEFEBQZIOKQ9DGGX","rating":"4","ts":"timestamp"},"RVRXUPMPKDMW99MKXGVLWDKQYX9FBFQFUSLPTQY9JWNZZMPMHLCQBPKSZCXPEBGLCGMYEJURZ9TPPITLI":{"vote":"XIXZPZFIBTWCYNJHCWEZGEKMDBYDXVXYQYJAHZ9Q9NQKCWGZCKJO9OVZPYOZGNUSYUUAPIXWGBEPZVNBW","rating":"4","ts":"timestamp"}}},"BZDAXEAZPFXCKTARAI9SBOEQP9M9TLXBUXEUPYUWNHZUNFIERKIKXPXLIOPNZGVYDMNBWCSBHYKCIJYHK":{"startdate":"2018:05:23:08:29:55","duration":"8","visitorAttest":"URIORVZFPTWUYNGMWV9KMVVZDVWLLFLYAUNARDVWIOQHJLMDCLGWLCK9BSOLAGFKGQWLDYUPQUMV99SEN","hotelierAttest":"GVXC9TUHCULK9WLSCLGLSJZDKKVTZZOXUXGKQMCPZCUOF9CNXRJ9SLZBKYXNPHRPDJVTESS99ZSGTFAUS"}}},"J9ROAZFQZJIAVEHCPZYJSZFD9VNKTYFFVUWXSACDESJLKCTUOGQ99XUTNZHHN9VYVDNP9ANJUFK9EYDLL":{"attestedRecipient":"did:discipl:iotaNKTISWGTAPOEQYCNGNKJCDYGQEAAOBRPS9PUKXGQIAQRRKZKQTKKOPPWEEAMDMRBFBVMYMVBLZOW9TNTC","data":{"NKTISWGTAPOEQYCNGNKJCDYGQEAAOBRPS9PUKXGQIAQRRKZKQTKKOPPWEEAMDMRBFBVMYMVBLZOW9TNTC":{"attestedEvent":"did:discipl:iotaFGZOITTRYLQVQKCZIGHPCSBFSKLCPOLEXFOXGZTEAZDJBRCPPWGJMBWMI9KZUAOEDCPZNXIELXMHCEDJZ","name":"My Event","description":"Party @ De Rotterdam","recipientAttest":"J9ROAZFQZJIAVEHCPZYJSZFD9VNKTYFFVUWXSACDESJLKCTUOGQ99XUTNZHHN9VYVDNP9ANJUFK9EYDLL","data":{"FGZOITTRYLQVQKCZIGHPCSBFSKLCPOLEXFOXGZTEAZDJBRCPPWGJMBWMI9KZUAOEDCPZNXIELXMHCEDJZ":{"index":null,"token":"TJ/FAbA5y4Mo+ZuNcyGM0qxLq2YdLkRAbUQOyIcb2P0hcLYCL0NhOan+LHHmRoEA","pubkey":"OjUL/MWutE+daT9JtODlyKB/V6GigR4Hx+HedaulywqV6g5NZgk74EzIVVloyYJPK7ag3U6JlhSVWrq2kmSlWA=="}}},"WDUPTUKYVTVWNYWIAFJWOXVC9JNFPAIRMWXNVLBMQORMUOQQHWFUFSIYKKTGHCUTQKDQMYEWOONNWH9FX":{"attestedEvent":"did:discipl:iotaVKFBOQZHZOJKVWBPXCIXETUBJMCFPEFYHWJKEQSVDJ9PGICCPSOACNSYXDYVVJYRMCSJD9OZVUJDXUYPO","name":"My Event","description":"Party @ De Rotterdam","recipientAttest":"J9ROAZFQZJIAVEHCPZYJSZFD9VNKTYFFVUWXSACDESJLKCTUOGQ99XUTNZHHN9VYVDNP9ANJUFK9EYDLL","data":{"VKFBOQZHZOJKVWBPXCIXETUBJMCFPEFYHWJKEQSVDJ9PGICCPSOACNSYXDYVVJYRMCSJD9OZVUJDXUYPO":{"index":null,"token":"5yxYTgiVfShsvYINFr2WgQLnKbYx4jp4AM8AMHD9gvqbFg1t1VEYWCT7CXjsf3Y7","pubkey":"MhCAsGLgK9mb4OXhbgHnCc2zWnE65nmuT2v8uq7Q3pnK63GKTmOd7JDhUXqnI3w0n4XJCUauE00+zsVVcPLz2A=="}}}}},"UMKFRCAWTXVRWBXBTCFXDGEXDBAMZOLZBMJUBBDLUGYDIJUFYHSNUK9JV9QIJKMKQD9UWGYJXQNYDPIJJ":{"attestedHotelier":"did:discipl:iotaYLETKEQYMZRQKWQQGDPLRBINPXMOGJAEB9WWGWUPRCXLLMOATZS9SZTYTGXDBWTHQDXPDECOEYNMKEWIC","data":{"YLETKEQYMZRQKWQQGDPLRBINPXMOGJAEB9WWGWUPRCXLLMOATZS9SZTYTGXDBWTHQDXPDECOEYNMKEWIC":{"attestedVisitor":"did:discipl:iotaKJLMT9XXWXQEXYFKDDZLGYYUYMEFDGRSYS9JD9RTAYQFYYYEKXYQBE9AGVAHOLTYXEUOVQMMDTPHZTNFT","data":{"KJLMT9XXWXQEXYFKDDZLGYYUYMEFDGRSYS9JD9RTAYQFYYYEKXYQBE9AGVAHOLTYXEUOVQMMDTPHZTNFT":{"attestedStay":"RXEGJTEKTWJL9PSRNGOECGGWWJRZISMYEDSCXHQHBVHZNIRDVJCBHTZTDRJODURORJMQ9EKDMMPXSCYZQ"},"LFFS9WNTOLNKLIII9ZYLMOBXEFF9ZONXMDCUGUEQHGLEJEEBQHYSCKEQBNMEU9WPWWQLAKRXSJBSRPQFM":{"vote":"HFWYYSRZLPZFQZESIRPEFWJPXKQVKDOC9GFHXKVMS9PN9LMNIJXDHGNGLTPIBYNCCOPINLCWUCAVGAJMD","rating":"1","ts":"timestamp"}}},"RXEGJTEKTWJL9PSRNGOECGGWWJRZISMYEDSCXHQHBVHZNIRDVJCBHTZTDRJODURORJMQ9EKDMMPXSCYZQ":{"startdate":"2018:05:23:08:36:20","duration":"4","visitorAttest":"YLETKEQYMZRQKWQQGDPLRBINPXMOGJAEB9WWGWUPRCXLLMOATZS9SZTYTGXDBWTHQDXPDECOEYNMKEWIC","hotelierAttest":"UMKFRCAWTXVRWBXBTCFXDGEXDBAMZOLZBMJUBBDLUGYDIJUFYHSNUK9JV9QIJKMKQD9UWGYJXQNYDPIJJ"}}},"OUXAROFNYP9BIMMIVTMISEOEIROKMTVVYMYWVMXFZRUUTTDGGXQZTRYHKXUFNOTXDHTPJPDEOOIPZBF9V":{"attestedRecipient":"did:discipl:iotaJFWXH9UJIUCZQRV9QRHQWGKLMQMNBLGBTGYTQWUQAVXIJEDZGJCPWPCFCNBHLTXDHYNKBZVITEHHJXGWG","data":{"JFWXH9UJIUCZQRV9QRHQWGKLMQMNBLGBTGYTQWUQAVXIJEDZGJCPWPCFCNBHLTXDHYNKBZVITEHHJXGWG":{"attestedEvent":"did:discipl:iotaXBGEBZLLFXSYRAGNLPPKVKBMPCMIT9HVGQLGBHGBCNLJXEXUXQHYQNQNNAZDYXS9AGUZOATQBJQXQDM9O","name":"My Event","description":"Party @ De Rotterdam","recipientAttest":"OUXAROFNYP9BIMMIVTMISEOEIROKMTVVYMYWVMXFZRUUTTDGGXQZTRYHKXUFNOTXDHTPJPDEOOIPZBF9V","data":{"XBGEBZLLFXSYRAGNLPPKVKBMPCMIT9HVGQLGBHGBCNLJXEXUXQHYQNQNNAZDYXS9AGUZOATQBJQXQDM9O":{"index":null,"token":"RS660wHaiaddsG61SoWD5Jk1L/e4SSXheH1ArqtKcPQ7QnJ4MBtNaVuxF1FyPJTC","pubkey":"uVYaerUedAVbcLuryGV0MKQ7pb8rr7A8aWe9LUhzktqjjXZwcEvgj2j5L9igx/2QtNMrQTokbM03/XpSrKGe0A=="}}}}},"XWZSPMFEYEXFEAWMYIDKXEJMFYAFXPVWHECEX9JDNEIDVFHVVCFKJYODZNVEXB9HVHB9MQSJNRPHUU9ZL":{"attestedRecipient":"did:discipl:iota9HLZENDZ9FXYKMRQSTSLVDSNCLGWGEMGMRPNLQJXWGKDMHDNFGKACCDV9JCHARKLHIMYNNIICVNVYDVFZ","data":{}},"ENWRNAWATJFWACKOPDQSYCFMGBYGNCNJKGDSUYKNDFFUMATUKCH9WBBLFJSCZTDUROCGEJUUFGVMLSNLV":{"attestedHotelier":"did:discipl:iotaXBUBCZPULIZPSMXNGKI9FVOEPZWXEMTTGBQFINJDTYMPCPYHZHBK9JVCUGTZFCWTMGRVIYVQFDWPIPBYE","data":{"XBUBCZPULIZPSMXNGKI9FVOEPZWXEMTTGBQFINJDTYMPCPYHZHBK9JVCUGTZFCWTMGRVIYVQFDWPIPBYE":{"attestedVisitor":"did:discipl:iotaBQXERUZARCFBCTFZXWCHIVGOOVUDIFPJXKKYEWZSYLBECGPFKJLNBMBBKTIIWWZOGWPPRGORBAPMBPKNE","data":{"BQXERUZARCFBCTFZXWCHIVGOOVUDIFPJXKKYEWZSYLBECGPFKJLNBMBBKTIIWWZOGWPPRGORBAPMBPKNE":{"attestedStay":"9DJGZYWDOFWNUVKONZKWHWOGWCCPCQMLO9LPTNSHCJNDAPYENWXOKQGYYQTQ9RCDSWIVMIHVMNOD9YXNW"},"YX9OGDAIRRBZGDQVFHY9WXYPEUKWXDQNFTKB9MKFCHJR9PLBQHJZBUEAGIEUJXJZQUANYPHZTNUNKHBZI":{"vote":"EBPNPOXNQJZBUTGSOK9WVMPHKQICKEYHNSDSG9PREXOYNYZZQAQVIMLDDGNAHKVRUCMSIYGFCKRXGAIBY","rating":"2","ts":"timestamp"},"YBBJJVDELKJAINYDCKWQTZZOTYMZW9RULOKOBENGKAXRLHPKVFCRGV9SM9ZSCDAOLENTVG9UV9OXTDOCZ":{"vote":"EBPNPOXNQJZBUTGSOK9WVMPHKQICKEYHNSDSG9PREXOYNYZZQAQVIMLDDGNAHKVRUCMSIYGFCKRXGAIBY","rating":"4","ts":1528201766749},"EDPWUHWAKCOB9QBKFYAFRVMIRLJNVRGLMNVHYAXOJECSJPGUCHBEZVAD9LWC9WOBEAHRBZWPSPBIXYNHS":{"vote":"NJ9UIEGDJRLKC9QSUHEOVMGFNDKKBXFVLAQYEKLSYRZLNTWQOBFD9NIIVCPAIUBJJRGYFWVSEBIQRGISY","rating":"3","ts":1528201891237}}},"9DJGZYWDOFWNUVKONZKWHWOGWCCPCQMLO9LPTNSHCJNDAPYENWXOKQGYYQTQ9RCDSWIVMIHVMNOD9YXNW":{"startdate":"2018:06:04:09:04:11","duration":"3","visitorAttest":"XBUBCZPULIZPSMXNGKI9FVOEPZWXEMTTGBQFINJDTYMPCPYHZHBK9JVCUGTZFCWTMGRVIYVQFDWPIPBYE","hotelierAttest":"ENWRNAWATJFWACKOPDQSYCFMGBYGNCNJKGDSUYKNDFFUMATUKCH9WBBLFJSCZTDUROCGEJUUFGVMLSNLV"}}},"GZYZJZRLI9SSOJKVTWNJPD9WBXQTJQENQCCUXSKQRROGDXQIAFCWEOFVGUPDEDCJVI9IBB9QMHGAYUUMQ":{"attestedRecipient":"did:discipl:iotaIIRANGGGZA9LSPKNZCNSZMIYMYOOPZKKKSMQTZUXBENNFY9ZQGGQYMRUHTTNCWQVBZEMUUWRNDEQXMSQB","data":{"IIRANGGGZA9LSPKNZCNSZMIYMYOOPZKKKSMQTZUXBENNFY9ZQGGQYMRUHTTNCWQVBZEMUUWRNDEQXMSQB":{"attestedEvent":"did:discipl:iota9JVPVMZYVZZAKCKDKXLSIAUXAO9MABGQHAJHONBRLZJYERHMVGIDUUUBPBYZXYSXTUQHESWMHBEROTCOH","name":"My Event","description":"Party @ De Rotterdam","recipientAttest":"GZYZJZRLI9SSOJKVTWNJPD9WBXQTJQENQCCUXSKQRROGDXQIAFCWEOFVGUPDEDCJVI9IBB9QMHGAYUUMQ","data":{"9JVPVMZYVZZAKCKDKXLSIAUXAO9MABGQHAJHONBRLZJYERHMVGIDUUUBPBYZXYSXTUQHESWMHBEROTCOH":{"index":null,"token":"a+QrfarqLxQW0YA9otK3nJNjjWDsM/yfksZ90DS0p9ru5qnjFr7Q6mOTuRE9dfWq","pubkey":"ydvwJK5OThI/6JsFomu0rKPLiD1OGURinKoTCrPVphJHnRCDaJrNVjU/ZzZbs8u6USTqG7Upip0tu+mrZLTYtw=="},"NTXDQVDGQUO9DIGWUDIPQLGITUHELQHWAZKGGIWOCKJBEWXIKYZQFXKFBDQ9BMIIWHVHO9HUGCVGOBSXN":{"index":null,"token":"aSmZHzjq3JoFlGfNjWFLMFSvmRV7mi/Y/pM6Uz8lwEDyR6KZCF+DbFwl9sh+a/I6","pubkey":"bGrcQJ5esWWToBazphJojMtkXZS5DVctzf9T0DPpsaxFfurIzMgpPrxp8puo/eZ9q0K7BjDPAT4zbll0QD1KEQ=="}}},"YDQMK9PCFITDBMYB9QCZTWOMOEIBVONWVIUFKMYHSJAKWKZGYKTHCABOIXNJSZYSVBMPNLPLORJELJEPC":{"attestedEvent":"did:discipl:iotaNEVRESDGNBZVJ99KDXAIBRKIKAMBWUFMKO9DMOXONSGNFBGEEVPXSNJAOQU9OM9JLGHGCRLKKUOV9XDNR","name":"My Event","description":"Party @ De Rotterdam","recipientAttest":"GZYZJZRLI9SSOJKVTWNJPD9WBXQTJQENQCCUXSKQRROGDXQIAFCWEOFVGUPDEDCJVI9IBB9QMHGAYUUMQ","data":{"NEVRESDGNBZVJ99KDXAIBRKIKAMBWUFMKO9DMOXONSGNFBGEEVPXSNJAOQU9OM9JLGHGCRLKKUOV9XDNR":{"index":null,"token":"MGKLl0dUiGur51NOFu9WueNXiLxWHGUtOVzcFOtwgGorIiWldwMh/veXOh0NIR4w","pubkey":"0aHBPL/Rz3k74FGcSzCtqzFpoufPsKgQ0dLuaE9PlxPO49pJygaP4qGRjMY1TCx3Po5/HKKuSLqgGkgzn5INeg=="}}}}},"FCKKBQRTEU9VTVKDMHK9DOQQAOICYNKVRIFLGLR9YYJKASUANLYBIJOFMXAXIXYJXMOYSWSMFPACOIRX9":{"attestedRecipient":"did:discipl:iotaGJESKSPQLZLGMLZWDYIT9YKTAOSNRCDAWMGBIXCLMJEC99LOOKPWCWJZEGUQKJWB9HZWLBWGTWYOQDEOR","data":{"GJESKSPQLZLGMLZWDYIT9YKTAOSNRCDAWMGBIXCLMJEC99LOOKPWCWJZEGUQKJWB9HZWLBWGTWYOQDEOR":{"attestedEvent":"did:discipl:iotaHXYWBXRVFYNNURY9ZGYHPNNRTFVHXDOQLDOAPNTPWFZJJM9IXPSAMNXVZSCMPPLJYWSRCRSBEEF9IWFAC","name":"My Event","description":"Party @ De Rotterdam","recipientAttest":"FCKKBQRTEU9VTVKDMHK9DOQQAOICYNKVRIFLGLR9YYJKASUANLYBIJOFMXAXIXYJXMOYSWSMFPACOIRX9","data":{}}}}}
```

###	`revoke`
Let's the actor belonging to the Ssid set in the given session effectively revoke claims it made earlier. Most times, the platforms revoke through additional claims denoting an earlier claim as being "revoked". You can not rely on effectively revoking such revoke-claims through this method. 

```
revoke(session, link)
```
