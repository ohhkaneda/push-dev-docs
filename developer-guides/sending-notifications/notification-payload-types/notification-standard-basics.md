# Notification Standard (Basics)

All notifications passed to the network are eventually transformed to JSON payload. The following glossaries help cover some of the basics that can help in understanding what is happening and how to customize the behavior.

* **Verification Proof**  - Each notification sent carries with itself a verification proof that allows the network to validate that the notification is coming from a channel or one of it's delegate.&#x20;

{% hint style="success" %}
Verification Proof is usually handled automatically by the Smart Contract or the SDK but you if you want to, you can read about it - [notification-verification-proof.md](notification-standard-advanced/notification-verification-proof.md "mention")
{% endhint %}

* **Notification Identity** - Each notification passed to the network is actually an identity which defines how the notification is formatted, where it's stored, etc.&#x20;

{% hint style="success" %}
For most part except for Smart Contracts, [push-sdk](../../../developer-tooling/push-sdk/ "mention")will abstract [notification-identity.md](notification-standard-advanced/notification-identity.md "mention")
{% endhint %}

* **Identity Type** - Defines the format in which the notification json payload is sent to the network. Common types you need to know about.

{% tabs %}
{% tab title="Identity Type" %}
| Identity Type | Remarks        | Remarks                        |
| ------------- | -------------- | ------------------------------ |
| 0             | Minimal        | Recommended for Smart Contract |
| 2             | Direct Payload | Recommended for PUSH SDK       |
{% endtab %}
{% endtabs %}

*   **Notification Type** - Defines the type of notification that is sent:

    * Broadcast (Type **1**) - Is sent out to all the users who have opted in to your channel
    * Targeted (Type **3**) - Is sent out to an individual user
    * Subset (Type **4**) - Is sent out to a subset of users of your channel


* **Notification Content** - Defines the notification content which consists of:
  * Notification JSON Object - What is shown on your home screen
  * Payload JSON Object - What is shown and stored on your feed
  * Recipients - 0x0 for type 1 (broadcast), 0xTarget for type 3 (Targetted) and \[0x01, 0x02, 0x03, ..., 0xN] for type 4 (Subset)&#x20;

```json
// Example Raw Content for targeted notifcation, abstracted away by SDK
{
  "notification": {
    "title": "The title of your message displayed on screen (50 Chars)",
    "body": "The intended message displayed on screen (180 Chars)"
  },
  "data": { // Is defined as payload most of the time
    "type": "3" // notification type
    "sectype": null // enables encrypted notifications
    "asub": "[Optional] The subject of the message displayed inside app (80 Chars)",
    "amsg": "[Optional] The intended message displayed inside app (500 Chars)",
    "acta": "[Optional] The cta link parsed inside the app",
    "aimg": "[Optional] The image url which is shown inside the app",
    "etime": "[Optional] if given, notif will be deleted after this in epoch"
    "hidden" :"[Optional] if given, notif will not show in user feed"
  },
    "recipients": 0xtarget
}
```

{% hint style="success" %}
These concepts are for your understanding as most of them are abstracted away but if you wish to dive deeper then [notification-payload.md](notification-standard-advanced/notification-payload.md "mention")
{% endhint %}

* **Content Markdown** - Defines how the content markdown should be passed to enable variety of notifications. Covered in the advanced section - [notification-content.md](notification-standard-advanced/notification-content.md "mention")\

* **Sender** - Defines who is sending the message. The sender is the channel address or the alias address. The address is represented in CAIP-10 format. For now, PUSH protocol supports Ethereum and Polygon chain. Their respective CAIP-10 format can be represented as:\

  * Ethereum(Goerli): `eip155:42:<Address>`
  * Polygon(Mumbai): `eip155:80001:<Address>`&#x20;

{% hint style="warning" %}
One exception to CAIP-10 format is for smart contract to smart contract interaction (ie: [using-smart-contract.md](../using-smart-contract.md "mention") [#smart-contract-example](notification-standard-basics.md#smart-contract-example "mention")), For this specific feature, the native blockchain address is required and not CAIP-10.
{% endhint %}

* **Source** - The source from which the notification is coming from. Currently supports `ETH_TEST_KOVAN`, `ETH_MAINNET`, `POLYGON_MAINNET`, `POLYGON_TEST_MUMBAI`, `THE_GRAPH`.\
  \
  Source is determined by the network it is coming from in case of smart contracts, internally for the graph and by the CAIP-10 format of the Sender for Push SDK.&#x20;

{% hint style="success" %}
Source is abstracted away unless you are interacting directly with Push Nodes, it's verified mostly through verification proof on push nodes to ensure it can't be spoofed.
{% endhint %}

* **Recipient(s)** - The address to the notification should reach. The address is represented in CAIP-10 format. For now, Push supports Ethereum and Polygon chain. Their respective CAIP-10 format can be represented as:\

  * Ethereum(Goerli): `eip155:42:<Address>`
  * Polygon(Mumbai): `eip155:80001:<Address>`&#x20;

{% hint style="warning" %}
One exception to CAIP-10 format is for smart contract to smart contract interaction (ie: [using-smart-contract.md](../using-smart-contract.md "mention") [#smart-contract-example](notification-standard-basics.md#smart-contract-example "mention")), For this specific feature, the native blockchain address is required and not CAIP-10.
{% endhint %}

### Examples of what you will be playing with

{% tabs %}
{% tab title="Smart Contract Example" %}
While any Notification Identity can be passed in any of the interactions, It's recommended to start with Identity Type 0 (Minimal) for smart contracts.\
\
**Format:** `0+<Notification Type>+<Title>+<Body>`\


**What to call:** `sendNotification(address _channel, address _recipient, bytes calldata _identity)`\
\
**Additional Rules:**

* Notification Type 1 (Broadcast): Pass _recipient as \_channel_
* Notification Type 3 (Targeted): Pass recipient as intended recipient
* Notification Type 4 (Subset): **Not Supported Yet**\


**Example:**&#x20;

```solidity
IPUSHCommInterface(EPNS_COMM_CONTRACT_ADDRESS_FOR_SPECIFIC_BLOCKCHAIN).sendNotification(
    YOUR_CHANNEL_ADDRESS, // from channel - recommended to set channel via dApp and put it's value -> then once contract is deployed, go back and add the contract address as delegate for your channel
    to, // to recipient, put address(this) in case you want Broadcast or Subset. For Targetted put the address to which you want to send
    bytes(
        string(
            // We are passing identity here: https://docs.epns.io/developers/developer-guides/sending-notifications/advanced/notification-payload-types/identity/payload-identity-implementations
            abi.encodePacked(
                "0", // this is notification identity: https://docs.epns.io/developers/developer-guides/sending-notifications/advanced/notification-payload-types/identity/payload-identity-implementations
                "+", // segregator
                "3", // this is payload type: https://docs.epns.io/developers/developer-guides/sending-notifications/advanced/notification-payload-types/payload (1, 3 or 4) = (Broadcast, targetted or subset)
                "+", // segregator
                "Title", // this is notificaiton title
                "+", // segregator
                "Body" // notification body
            )
        )
    )
);
```
{% endtab %}

{% tab title="Push SDK Example" %}
Push SDK supports all Identity Types but it is recommended to use Identity Type 2 (Direct Payload) as it's blazingly fast!\
\
**What to call:** \
[push-sdk](../../../developer-tooling/push-sdk/ "mention")\
\
**Examples:**

```json
// apiResponse?.status === 204, if sent successfully!
const apiResponse = await EpnsAPI.payloads.sendNotification({
  signer,
  type: 1, // broadcast
  identityType: 2, // direct payload
  notification: {
    title: `[SDK-TEST] notification TITLE:`,
    body: `[sdk-test] notification BODY`
  },
  payload: {
    title: `[sdk-test] payload title`,
    body: `sample msg body`,
    cta: '',
    img: ''
  },
  channel: 'eip155:42:0xD8634C39BBFd4033c0d3289C4515275102423681', // your channel address
  env: 'staging' // 'staging' or 'prod'
});
```



```json
// apiResponse?.status === 204, if sent successfully!
const apiResponse = await EpnsAPI.payloads.sendNotification({
  signer,
  type: 3, // targeted
  identityType: 2, // direct payload
  notification: {
    title: `[SDK-TEST] notification TITLE:`,
    body: `[sdk-test] notification BODY`
  },
  payload: {
    title: `[sdk-test] payload title`,
    body: `sample msg body`,
    cta: '',
    img: ''
  },
  recipients: 'eip155:42:0xD8634C39BBFd4033c0d3289C4515275102423681', // recipient address
  channel: 'eip155:42:0xD8634C39BBFd4033c0d3289C4515275102423681', // your channel address
  env: 'staging'
});
```



```json
const apiResponse = await EpnsAPI.payloads.sendNotification({
  signer,
  type: 4, // subset
  identityType: 2, // direct payload
  notification: {
    title: `[SDK-TEST] notification TITLE:`,
    body: `[sdk-test] notification BODY`
  },
  payload: {
    title: `[sdk-test] payload title`,
    body: `sample msg body`,
    cta: '',
    img: ''
  },
  recipients: ['eip155:42:0xD8634C39BBFd4033c0d3289C4515275102423681', 'eip155:42:0xCdBE6D076e05c5875D90fa35cc85694E1EAFBBd1'], // recipients addresses
  channel: 'eip155:42:0xD8634C39BBFd4033c0d3289C4515275102423681', // your channel address
  env: 'staging'
});
```
{% endtab %}

{% tab title="Subgraph Example" %}
Subgraph are special case where the content is verified from the subgraph network (currently hosted one). Head to this section to understand more [using-subgraph-gasless.md](../using-subgraph-gasless.md "mention")
{% endtab %}
{% endtabs %}





\
\


{% tabs %}
{% tab title="Smart Contract" %}

{% endtab %}

{% tab title="Gra" %}

{% endtab %}
{% endtabs %}
