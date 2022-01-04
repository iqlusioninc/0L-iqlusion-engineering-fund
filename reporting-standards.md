# Reporting Standards for Community Wallets

The 0L ecosystem relies on Community Wallet operators to serve as stewards of the ecosystem by funding diverse activities. The goal of the Polycentric Programs is to ensure that there is an evolving and diverse set of stakeholders operating community wallets but in order for the 0L commmunity members to choose how and where to allocate resources there needs to be transparency in the allocation of community funds. Pursuant to creating transpareny this document outlines a minimal viable reporting standard for community wallets.

## Overview

![](https://i.imgur.com/q7M27AV.png)

## Motivating User Stories

As a community member I want to have visibility into where, when and why community wallets disperse funds.

As a community wallet operator, I need visibility into the counterparties recieving dispersements.

As a project contributor, I need to the right to decide how much or how information is attached to my identity within the community.

## Implementation Agnostic

The proposed solution is that each individual Community Wallet Operator will maintain a simple public record of their funding activities that satisfies the extensible schema presented in this document. 

The emphasis is on the flexible data structures because they serve as a kind of communication protocol. Human users and software systems alike can leverage this data structure to read and write information about the activities of within the ecosystem.

This *solution architecture* **neither** requires **nor** precludes the implementation of this reporting functionality into software. Furthermore, it enables but does not require that some of this software get deployed on the 0l blockchain.

### Minimal Implementation

Practically, an implementation is required. The minimal implementation of this standard is for the Community Wallet operators maintain copies of the relevant files `contributors.json` and `payments.json` in public github repos. Contributors may modify their own records in a `contributors.json` file via pull request. Community Wallet operators may commit changes to their `payments.json` by writing in new records representing new payments. The git history of the `payments.json` provides provenance for the data and should not be tampered with.

It is expected that extension to more sophisticated implementations will follow. Friction in reporting could be greatly reduced by a simple web app, but it is important to note that we do not wish to create centralization around that app. As noted above, this standard is intentionally agnostic of implementation.

## Contributors

The identity of a contributor in the 0L community is an onchain account. As that 0L network grows onchain accounts will develop an activity history. Inititally that history simply comprises the transaction that added it to the network and a tower height if it has chosen to mine. As the karma hustle ramps up, contributors will be reciving payments from community wallets sponsoring missions. Community wallet operators need to be able to match the wallets of recipients to discord usernames, github handles, or other online identities in order to ensure the funds are dispersed to the appropriate community members.

In order to ensure transparency in the stewardship of community funds the community wallet operators *may* require the contributor to provide some information. Strictly speaking the only required information is a recieving address, but it is expected that community wallet operators will due diligence to ensure that the compensated parties are actually the once who provided the labor. 

In order to ensure consistency across independently managed contributor database the `contributors.json` is as follows.

### Contributor Self-Reporting

The following top level schema is required. It provides the basic structure for joining records to accounts on chain without providing any strict requirements on what information is required.

```
{"account":<account>,
"type":<int>,
"details":<dict>}
```

It is nontheless possible to add addition fields to the top level record, as well as to define types assuming a master mapping is managed.


| Index | Type | Description | Required Fields| Optional Fields |
| -------- | -------- | -------- | -------| -------- |
| 0     | contributor     | this is the default type     | None | `'githubId', 'discordId', 'twitterId', 'gdocsId',...`|
| 1 | team | this type for use by collectives | None | `'name','description',members',...`|
| $\vdots$ | $\vdots$ | $\vdots$ | $\vdots$| $\vdots$|

This structure intentionally avoids required fields but leaves a provision for them as there may need to be some more detailed types which require *required fields* to satisfy APIs to external systems.

### Extensible Schemas

The extended schema for the default contributor type might look as follows.
```
{"account":<account>,
"type":0,
"details":{
    'githubId':<str>,
    'discordId':<str>,
    'twitterId':<str>,
    'gdocsId':<str>
    }
}
```
It is important to note that the information requested in details may vary widely depending on the community wallet operator and the kinds of contributions they are compensating. The example above is based on my specific experiences as a contributor. I have collaborated with other 0L contributors via github, discord, google docs and twitter. 
```
{"account":27de1f87599fd1f8ae5bd1101511d546,
"type":0,
"details":{
    'githubId':"mzargham",
    'discordId':"250086586450968576",
    'discordName':"mZ#3472",
    'twitterId':"mzargham",
    'gdocsId':"michael.zargham@gmail.com"
    }
}
```

I have made personal choices to use a specific social accounts which are directly identifiable to me personally, but I could just have easily contributed psuedonomously by creating new accounts in one or more of these platforms. Even if I had done so, this data model allow me to create a strong "local" identity reflecting my personal history as an 0L contributor. 

Another mode of collaboration that is common in online communities to form teams or working groups. Working groups can accomplish higher complexity tasks with more reliability by sharing the work, even when the exact roles and responsibilities are variable. By including teams in our contributors model, subsidiarity is supported naturally. The extended schema for a team contributor might look like the following.

```
{"account":<account>,
"type":1,
"details":{
    'name': <str>
    'description':<str>
    'pointOfContact':<account>,
    'multisig':<bool>,
    'members':<list of accounts>
    }
}
```
This pattern is appropriate for when working groups need their own budget allocations. The working group can have a well defined persistent identity even the invidual members or even the point of contact change over time. I have mocked up some additional fields that could help ensure greater transparency and accountability for team type contributors. Specifically, I am implying that teams should use multisig accounts, but they should still provide a `pointOfContact` who is an individual contributor account. They may or many not choose to provide a list of additional individual contributor accounts. 

## Payments

Payments are naturally represented on chain as transactions but they lack the relevant context to make them useful for accountability purposes. Community wallets in particular have already made their accounts public, as well as extensive details about their respective goals. The flow of funds to contributors can be identified by enriching the transactions with contributor metadata retreived from a `contributors.json` file. However, this doesn't provide context about specifically activies were funded by this transfer. The purpose of the `payments.json` is to provide that context but this standard must support an evolving variety of use cases. Therefore, extensible schema approach is again employed.

```
{"transaction":<transactionHash>,
"type":<int>,
"details":<dict>}
```

### Relationship to onchain data

Note that the following details are not needed in the `payments.json` record precisely because they are available via any block explorer. It is expected that database-based implementations of this standard will leverage theopen source block explorer code to collect and join this information for reporting purposes.

```
{"transaction":<transactionHash>,
"sender":<account>,
"recipient":<account>,
"amount":<account>,
"epoch":<int>,
"round":<int>
```

As with the contributor schema, we are intentionally keeping the top level schema generic and emphasizing substandarization of types.

| Index | Type | Description | Required Fields| Optional Fields |
| -------- | -------- | -------- | -------| -------- |
| 0     | recievable  | this is the default type     | None | `'memo',...`|
| 1 | bounty | this type for paying mission bounties | None | `'memo','missions',...`|
| $\vdots$ | $\vdots$ | $\vdots$ | $\vdots$| $\vdots$|

The default type "recievable" is intentionally generic. Although, the memo field is not required, absense its use there is scarcely any point to recording the the transaction in `payments.json`. However, it is up to the community to hold community wallet operators to a high standard of reporting. If a community wallet is dispersing funds with at least reporting mimimal memos, then it behooves us to reallocate our resources towards community wallets that meet our reporting expectations.

### Extensible Schemas

Based on the current activies in the 0L ecosystem, we're expecting community wallets to primarily fund bounties for missions developed as part of karma hustle. An extended schema for a bounty payment for a mission or collection of missions would look like.

```
{
"transaction":<transactionHash>,
"type":1,
"details":
    {
    "memo": <str>,
    "missions":
        [
            {
            'recordURL':<URI>,
            'author':<account>,
            'worker':<account,
            'reviewer':<account>,
            'artifacts':<list of URIs>
            },
            ...,
            'recordURL':<URI>,
            'author':<account>,
            'worker':<account,
            'reviewer':<account>,
            'artifacts':<list of URIs>
            }
        ]
    }
}
```

Note that in the above I have further substandardized by proposing a data structure for specifically encoding a batch of missions that were completed. While not all bounties will have more than one mission, in practice missions are broken down into small well defined tasks. However, it is extremely common for these tasks to be interelated, and for them to be defined, completed and reviewed in clusters.

As it happens this standard is being proposed and dogfooded by the analytics working group as part our effort to increase transparency in the 0L ecosystem. The following example transaction record is a prototype of the first payment to be disbursed by the Deep Technology Research Program\<link\>.  Here is an example record.

```
{
"transaction":<0xTBD>,
"type":<1>,
"details":
    {
    "memo": "Karma Hustle bounty for completing the first batch of missions in the analytics project, specifically making permission tree data accessible.",
    "missions":
        [
            {
            'recordURL':'https://airtable.com/appuiDshjiBzOAGl2/tblixhNzkEaKt3a75/viwlz7S2kmgCkoHFS/rec4D7I4k2KS4ytt5',
            'author':<mZ's account here>,
            'worker':<gnuAndrew's account here>,
            'reviewer':<B's acccount here>,
            'artifacts':[<github link to code>]
            },
            ...,
            'recordURL':'https://airtable.com/appuiDshjiBzOAGl2/tblixhNzkEaKt3a75/viwlz7S2kmgCkoHFS/recXtnhbiUpaBMKQx?blocks=hide',
            'author':<mZ's account here>,
            'worker':<gnuAndrew's account here>,
            'reviewer':<B's acccount here>,
            'artifacts':[<github link to code>]
            }
        ]
    }
}
```

## Protoype

The prototype implementation of `contributors.json` and `payments.json` can be found at \<link to github\>, which at the time of this document contains the original three contributors to the analytics project and the first payment by the Deep Technology Research Program Community wallet with on chain address \<address\>. The associated transaction hash and its details can be found on the block explorer \<link to tx on block explorer\>.
    
We propose the following next steps:
* Other Community wallets should fork our `contributors.json` and `payments.json` files to seed there own versions of these files.
* after a few weeks of testing out the data structures proposed herein community wallet operators convene to discuss and refine this reporting standard with input from community
* assuming this standard or variation thereof is deemed suitable, proceed with the development of MVP software to streamline data entry and provide basic reporting functionality