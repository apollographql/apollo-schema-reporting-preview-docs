# Preview: Apollo Schema Reporting

Welcome to our pre-release documentation for Apollo Schema Reporting, **a protocol and implementation for GraphQL servers to automatically report their schema to the Apollo registry**. 

This repository is meant to provide early access users with information about this new project and instructions to configure schema-reporting, along with a central place to provide feedback to the schema reporting team.

## Before you start: IMPORTANT DISCLAIMER

The documentation here is currently only a preview of pre-release software. You may use it at will, but this evaluation product is not yet fully supported or considered generally available. While we will try to avoid breaking changes once a feature is in use, until the features discussed herein are taken out of preview we could make breaking changes. Please be aware of the heightened risk of using preview software.

Use of anything described in these preview docs (current as of April 2020) is done at your own risk and is governed by [The Apollo Terms of Service](https://www.apollographql.com/Apollo-Terms-of-Service.pdf).

PREVIEWS ARE PROVIDED "AS-IS," "WITH ALL FAULTS," AND "AS AVAILABLE," AND ARE EXCLUDED FROM ANY SERVICE LEVEL AGREEMENTS AND LIMITED WARRANTY. Previews may not be covered by customer support. Previews may be subject to reduced or different security, compliance and privacy commitments, as further explained in the Terms of Service, Privacy Policy and any additional notices provided with the Preview. We may change or discontinue Previews at any time without notice. We also may choose not to release a Preview into "General Availability."

## Getting started

If you're reading this, you've discovered, either through communication with our team or scouring docs & changelog entries, that we are exploring reporting schema definitions from running servers. This functionality is designed to make it easy and accurate to track the evolution of your GraphQL schema over time, without needing to make changes to your deployment pipeline.

This documentation will cover a few different components of what we're working on in more detail, including:

1. The protocol for schema reporting
2. The Apollo Server reference implementation of schema reporting
3. Automatic schema promotion to the registry

This top-level README will provide an overview, along with information on how to get started today with Apollo Server.

## Overview

Apollo schema reporting is a GraphQL API, by which running servers can report the shape of the schema they're serving to Apollo Graph Manager, in the form of a [GraphQL document (SDL)](https://www.apollographql.com/docs/apollo-server/schema/schema/#the-schema-definition-language). A running GraphQL server needs only have this document and an identifier for this document in order to report its schema. You must use a `service` token to configure a GraphQL server to report its schema, and optionally specify a particular `variant` (e.g. "staging").

Once a server (or a fleet of server instances) are reporting schemas, Apollo Graph Manager will update the schema registered to a graph's variant to the most recently reported schema from that fleet. When multiple schemas are reported simultaneously, Apollo Graph Manager will use its "automatic promotion" algorithm to choose which schema should be registered.

## Configuring Apollo Server

In order to configure Apollo Server **(v2.18+)** to report schemas, you will need to configure Apollo Server with a `service` API token. For those of you who have already configured metrics reporting from Apollo Server, you have already passed this token to your server and can skip this step. This token can be found in the graph settings page of [Apollo Graph Manager](https://engine.apollographql.com) for your graph. To pass this token to Apollo Server, simply set the `APOLLO_KEY` environment variable.  See [the docs](https://www.apollographql.com/docs/graph-manager/setup-analytics/) for more information.

Additionally, you will need to opt in to the schema-reporting feature. You can do this either with an environment variable:

```sh
APOLLO_SCHEMA_REPORTING=true
```

or via constructor options passed to Apollo Server

```js
new ApolloServer({
  engine: {
    experimental__schemaReporting: true,
  },
  ...
});
```

This is all you need to set up your Apollo Server instance to report its schema to Apollo Graph Manager!

Additionally, you can optionally specify a variant via the `APOLLO_GRAPH_VARIANT` environment variable.

```sh
APOLLO_GRAPH_VARIANT=production
```

If you do not specify a variant, the default variant `current` will be used for reporting.

## Automatic Promotion (in Apollo Graph Manager)

Once a server (or a fleet of server instances) are reporting schemas, Apollo Graph Manager will update the **active** schema for a graph's variant to the most recently reported schema from that fleet.

If server instances are reporting multiple different schemas simultaneously (e.g. during a deployment), Apollo Graph Manager will determine promotion rules using the following algorithm:

1. If there is no schema active for the graph variant, choose the most recently reported schema
2. If there is a schema active, and that schema has been reported in the last 60 seconds (not configurable), do nothing
3. If there is a schema active, and that schema has not been reported in the last 60 seconds, choose the most recently reported schema
4. Otherwise, do nothing

## Non-Apollo Server runtimes

If you are interested in support for schema reporting to Apollo Graph Manager from a non-Apollo server runtime, we would love to hear from you and help provide support in implementing the protocol & integrating with our schema-reporting functionality. Please open an issue on this repository to indicate your interest, including which server runtime you would like to see supported.