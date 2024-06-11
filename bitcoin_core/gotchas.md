# "Gotchas" when Onboarding
Below is a running list of "gotchas" that one may want to look out for when onboarding into Bitcoin Core.  As more info is gleamed over time, references will be added.

## Historical Context
Bitcoin development is distributed, not centralized, so there is no strict project management structure, and no formal training process.  Sometimes it's difficult to have all the historical context.  For example, in [PR 29530](https://github.com/bitcoin/bitcoin/pull/29530#issuecomment-1978887560) sipa had the historical context of an intent to remove the concept of a misbehavior/ban score and the last steps in doing this weren't finished.

https://bitcoinops.org/ provides some recaps and history.  Not sure if there are other resources available that would help onboarders avoid retreading past ground.

Perhaps increased documentation in the `doc` portion of the repo, specific to the functional area of the design, could help.  For example, a design document for P2P might include the current design approach as well as a "historical decisions" or "deprecated approaches" section.  A risk being that documentation and actual code get out of sync.

## Rebasing
Don't perform extraneous rebases.  A rebase invalidates existing ACKs, so once these exist it should only be done if there is a conflict (Drahtbot will complain) or a specific reason (such as a silent conflict). The CI will always rebase on master anyway for its runs (which could be restarted without rebasing if really necessary).

## Differences between Debug and Production Builds
Some methods behave differently in debug vs production builds.
For example, the `Assume()` macro behaves like `Assert()` in debug builds:

https://github.com/bitcoin/bitcoin/blob/5bc9b644a4b6af15a10ebd791c11d9340d01957f/src/util/check.h#L76-L89
```c++
/** Identity function. Abort if the value compares equal to zero */
#define Assert(val) inline_assertion_check<true>(val, __FILE__, __LINE__, __func__, #val)

/**
 * Assume is the identity function.
 *
 * - Should be used to run non-fatal checks. In debug builds it behaves like
 *   Assert()/assert() to notify developers and testers about non-fatal errors.
 *   In production it doesn't warn or log anything.
 * - For fatal errors, use Assert().
 * - For non-fatal errors in interactive sessions (e.g. RPC or command line
 *   interfaces), CHECK_NONFATAL() might be more appropriate.
 */
#define Assume(val) inline_assertion_check<false>(val, __FILE__, __LINE__, __func__, #val)
```

https://github.com/bitcoin/bitcoin/blob/5bc9b644a4b6af15a10ebd791c11d9340d01957f/src/httprpc.cpp#L76-L80
```c++
static void JSONErrorReply(HTTPRequest* req, UniValue objError, const JSONRPCRequest& jreq)
{
    // Sending HTTP errors is a legacy JSON-RPC behavior.
    Assume(jreq.m_json_version != JSONRPCVersion::V2);

```

In the above, when sending a numerical method (instead of string method) in JSON RPC, in debug builds, bitcoind aborts (stops).  For example, `curl --user $(cat ~/.bitcoin/regtest/.cookie) --data-binary '{"jsonrpc":"2.0", "method":2}' -H 'content-type: text/plain;' http://127.0.0.1:18443/`
In production builds, this does not happen (e.g. instead receive `{"result":null,"error":{"code":-32600,"message":"Method must be a string"},"id":"tester"}`).  It's important to recognize this distinction, because events that would not impact production builds may present as issues (e.g. a DoS).
