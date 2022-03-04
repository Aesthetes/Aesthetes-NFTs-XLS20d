# Aesthetes-NFTs-XLS20
### This is how we mint NFTs on XRPL XLS20 Testnet

For this new type of asset, we wanted to give users complete freedom of choice for the hosting platform of their digital content.  
For this purpose, we will leverage the URI field contained inside each NFToken for content referencing, as it allows up to 256 bytes of data to be stored in it.  
The URI field has to be formatted as a '\n' separated list of "key:value" formatted strings, for example

```
key1:value1\n
key2:value2\n
...
```
The best way to do this is to completely decouple the content's identifiers and the paths used to host it by storing only content identifiers inside the URI field and then constructing complete links by using some issuer-controlled source.  
One of the "key:value" pairs could as well be a complete link, but this would lose one of the biggest advantages of this protocol: the ability to migrate the content from one platform to another without invalidating the token.  
Our proposed solution is to use the already established .toml file schema: it's better if the wallet issuing the NFT is managed by a well-known entity which exposes a .toml file at a well-known location, actually at https://DOMAIN/.well-known/xrp-ledger.toml where DOMAIN is the value of the issuer account's Domain field.  
This file will be used for verifying the well-known entity's ownership of the XRPL wallet issuing the NFT and for extracting a *URI resolution list*.  
The URI resolution list is a list of incomplete links with placeholders in it. Each placeholder will be replaced by taking the value having the corresponding key inside the URI field of the NFToken, thus constructing a complete link.  
In the eventuality that no valid URI resolution list is extracted from the .toml file exposed by the issuer or if the issuer wallet does not have a value inside its "Domain" field which points to a valid .toml file, a default URI resolution list is taken and used, which is the one [here](https://xrpl.aesthetes.art/.well-known/xrp-ledger.toml).  
Besides the "key:value" pairs contained in the URI, some additional pairs will be extracted from the NFToken itself:
* Issuer, the address of the wallet that minted the NFToken.
* TokenTaxon, an additional field introduced by the XLS-20d proposal.
* TokenSeq, the serial number of the NFT.
* TokenID, an identifier calculated starting from the three fields described above.

For explanation of how these fields are handled please refer to the [XLS-20d standard proposal](https://github.com/XRPLF/XRPL-Standards/discussions/46).  
A quick note about the TokenTaxon field: for now the field must remain at zero as in the future it will be used to identify collections, with the value zero indicating an NFT with no collection related.  
Furthermore, special treatment is reserved for the "path" field: usually, if an incomplete link can't be completed by substituting all of its placeholders it is considered as invalid and so is discarded.  
In the case of "path", instead, it's simply ignored, or treated as if it was present and was void.

As we chase IPFS as our storing solution, if you are just an artist or content creator and you don't want to buy a domain only for storing the .toml file we suggest to use IPFS as well, by placing the "cid:IPFS_CID" string inside the URI.  
The links will be then completed by leveraging the URI resolution list found [here](https://xrpl.aesthetes.art/.well-known/xrp-ledger.toml).  
For example, if you place inside the URI

```
cid:QmQpLBHB6YSRYozVMbfLgB9YgPZaush5bYbESgoPcWqdXc
```
and the URI resolution list is

``` 
[URI_RESOLUTION]
list = [
  "https://ipfs.infura.io/ipfs/{:cid:}{:path:}",
  "https://gateway.pinata.cloud/ipfs/{:cid:}{:path:}",
  "https://ipfs.io/ipfs/{:cid:}{:path:}",
]
```
then the complete links will be

```
https://ipfs.infura.io/ipfs/QmQpLBHB6YSRYozVMbfLgB9YgPZaush5bYbESgoPcWqdXc
https://gateway.pinata.cloud/ipfs/QmQpLBHB6YSRYozVMbfLgB9YgPZaush5bYbESgoPcWqdXc
https://ipfs.io/ipfs/QmQpLBHB6YSRYozVMbfLgB9YgPZaush5bYbESgoPcWqdXc
```
These links will be tried in order for the retrieval of a json formatted text file that will be taken as the metadata file.  
The link from which the file is successfully retrieved is very important, since if this is not included inside the list of *immutable sources* then the data is considered non-immutable (so also the NFT).  
The list of immutable sources is a list of regular expressions, which can be found, again, [here](https://xrpl.aesthetes.art/.well-known/xrp-ledger.toml), under the name "IMMUTABLE_SOURCES".  
If you don't want to use one of the immutable sources to host your file while still wanting to guarantee a verifiable proof of immutability, you can leverage the special URI field "0x64_SHA256".  
This field is used, aside for URI resolution, for comparing the value contained in it with the freshly calculated SHA256 of the retrieved file. If the two match, then the retrieved data is considered as immutable, otherwise the content of the file has changed or is not the one which the issuer claims it is.  
In particular, all the fields starting with the "0x" are treated as hex formatted and taken as they are and not converted to text. They must come under a key formatted as "0xlength_name", with "length" being the (decimal) number of hexadecimal digits included in the URI field and "name" being the identificator of the field.  
This is done in order to save space in space-constrained situations like, for example, the URI field of an NFToken, where normal utf8-encoded text has to be converted into hexadecimal format when it's written on the blockchain, thus doubling the number of digits needed.

The structure currently used for the metadata file is not the final one, as we will consider the integration of major XRPL standards for this type of file when they will be official.  
For now, this file is composed of the following fields:
* **name**: the title of the content referenced by this file.
* **author**: the author of the content referenced by this file. It's an object with a "type" field that tells applications how to use it. For now, only the "Simple" type has been defined which contains the name of the author, maybe in future it will host other information as well.
* **description**: a brief description for the content referenced by this file.
* **issuer**: used for double checking, it contains an XRPL address. It has to be the same of the issuer of the NFToken refencing this metadata file, otherwise someone else could create an NFT and point it toward this file to trick users into buying a replica of the original NFT.
* **seqnum**: used for double checking, it must contain the same value of the "nft_serial" property of the on-chain NFToken. This is used together with the issuer address to uniquely identify an NFT.
* **content**: a string following the same "key:value" format of the URI field which points to a multimedial content. The procedure for constructing a complete link is the same as the one used for the metadata file.

Since a lot of discussion is happening around this topic, we want to emphasize the fact that this is not the final state of the protocol.  
We will keep an eye out on the community for proposals that can improve this protocol and we will carefully consider any suggestion.  
For now, you can enjoy our [NFT visualizer](https://xls20d.xrplnft.art/) running on the XLS-20 Testnet.
