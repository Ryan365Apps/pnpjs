# @pnp/graph : behaviors

The article describes the behaviors exported by the `@pnp/graph` library. Please also see available behaviors in [@pnp/core](../core/behaviors.md), [@pnp/queryable](../queryable/behaviors.md), [@pnp/sp](../sp/behaviors.md), and [@pnp/nodejs](../nodejs/behaviors.md).

## DefaultInit

The `DefaultInit` behavior, itself a composed behavior includes Telemetry, RejectOnError, and ResolveOnData. Additionally, it sets the cache and credentials properties of the RequestInit and ensures the request url is absolute.

```TypeScript
import { graphfi, DefaultInit } from "@pnp/graph";
import "@pnp/graph/users";

const graph = graphfi().using(DefaultInit());

await graph.users();
```

## DefaultHeaders

The `DefaultHeaders` behavior uses InjectHeaders to set the Content-Type header.

```TypeScript
import { graphfi, DefaultHeaders } from "@pnp/graph";
import "@pnp/graph/users";

const graph = graphfi().using(DefaultHeaders());

await graph.users();
```

> DefaultInit and DefaultHeaders are separated to make it easier to create your own default headers or init behavior. You should include both if composing your own default behavior.

## Endpoint

This behavior is used to change the endpoint to which requests are made, either "beta" or "v1.0". This allows you to easily switch back and forth between the endpoints as needed.

```TypeScript
import { graphfi, Endpoint } from "@pnp/graph";
import "@pnp/graph/users";

const beta = graphfi().using(Endpoint("beta"));

const vOne = graphfi().using(Endpoint("v1.0"));

await beta.users();

await vOne.users();
```

It can also be used at any point in the fluid chain to switch an isolated request to a different endpoint.

```TypeScript
import { graphfi, Endpoint } from "@pnp/graph";
import "@pnp/graph/users";

// will point to v1 by default
const graph = graphfi().using();

const user = graph.users.getById("{id}");

// this only applies to the "user" instance now
const userInfoFromBeta = user.using(Endpoint("beta"))();
```

Finally, if you always want to make your requests to the beta end point (as an example) it is more efficient to set it in the graphfi factory.

```TypeScript
import { graphfi } from "@pnp/graph";

const beta = graphfi("https://graph.microsoft.com/beta");
```

## GraphBrowser

A composed behavior suitable for use within a SPA or other scenario outside of SPFx. It includes DefaultHeaders, DefaultInit, BrowserFetchWithRetry, and DefaultParse. As well it adds a pre observer to try and ensure the request url is absolute if one is supplied in props.

The baseUrl prop can be used to configure the graph endpoint to which requests will be sent.

> If you are building a SPA you likely need to handle authentication. For this we support the [msal library](../concepts/authentication.md#MSAL-in-Browser) which you can use directly or as a pattern to roll your own MSAL implementation behavior.

```TypeScript
import { graphfi, GraphBrowser } from "@pnp/graph";
import "@pnp/graph/users";

const graph = graphfi().using(GraphBrowser());

await graph.users();
```

You can also set a baseUrl. This is equivelent to calling graphfi with an absolute url.

```TypeScript
import { graphfi, GraphBrowser } from "@pnp/graph";
import "@pnp/graph/users";

const graph = graphfi().using(GraphBrowser({ baseUrl: "https://graph.microsoft.com/v1.0" }));

// this is the same as the above, and maybe a litter easier to read, and is more efficient
// const graph = graphfi("https://graph.microsoft.com/v1.0").using(GraphBrowser());

await graph.users();
```

## SPFx

This behavior is designed to work closely with SPFx. The only parameter is the current SPFx Context. `SPFx` is a composed behavior including DefaultHeaders, DefaultInit, BrowserFetchWithRetry, and DefaultParse. It also replaces any authentication present with a method to get a token from the SPFx aadTokenProviderFactory.

```TypeScript
import { graphfi, SPFx } from "@pnp/graph";
import "@pnp/graph/users";

// this.context represents the context object within an SPFx webpart, application customizer, or ACE.
const graph = graphfi().using(SPFx(this.context));

await graph.users();
```

If you want to use a different form of authentication you can apply that behavior after `SPFx` to override it. In this case we are using the [client MSAL authentication](../msaljsclient).

```TypeScript
import { graphfi, SPFx } from "@pnp/graph";
import { MSAL } from "@pnp/msaljsclient";
import "@pnp/graph/users";

// this.context represents the context object within an SPFx webpart, application customizer, or ACE.
const graph = graphfi().using(SPFx(this.context), MSAL({ /* proper MSAL settings */}));

await graph.users();
```

## Telemetry

This behavior helps provide usage statistics to us about the number of requests made to the service using this library, as well as the methods being called. We do not, and cannot, access any PII information or tie requests to specific users. The data aggregates at the tenant level. We use this information to better understand how the library is being used and look for opportunities to improve high-use code paths.

> You can always opt out of the telemetry by creating your own default behaviors and leaving it out. However, we encourgage you to include it as it helps us understand usage and impact of the work.

```TypeScript
import { graphfi, Telemetry } from "@pnp/graph";
import "@pnp/graph/users";

const graph = graphfi().using(Telemetry());

await graph.users();
```
