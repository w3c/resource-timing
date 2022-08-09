# Render Blocking Status

Proposal to add a new field `render blocking status` to PerformanceResourceTiming which is an [enum](https://webidl.spec.whatwg.org/#idl-enums)  highlighting the status of stylesheets and scripts.

## Use cases

RUM providers could then easily determine the render blocking resources without having to rely on complex heurestics to determine the same.
This would enable analysis for below scenaarios and possibly more.
- Determine resources downloaded before FCP but were not render blocking? (and hence, could've been delayed)
- Determine resources that were render blocking, but weren't discovered early enough? (and hence, could've benefited from being preloaded)
(Reference : https://github.com/w3c/resource-timing/issues/262)


## API Changes and Example Code

The PerformanceResourceTiming Interface in <a href="https://w3c.github.io/resource-timing/#sec-performanceresourcetiming">resource-timing</a> would be updated to 
```bash
[Exposed=(Window,Worker)]
interface PerformanceResourceTiming : PerformanceEntry {
    ...
    ...
    readonly attribute RenderBlockingStatusType renderBlockingStatus;
    ...
    ...
    [Default] object toJSON();
};

enum RenderBlockingStatusType {
    "blocking",
    "non-blocking"
};
```

Sample usage:
```javascript
const entry_list = performance.getEntriesByType("resource");
console.log(entry_list[0].renderBlockingStatus);
```


## Render Blocking Status Values

The following values are proposed for the resource blocking status enum

`blocking` - A potentially render blocking resource

`non-blocking` - Non blocking resource

We primarily reuse the value of the [render blocking](https://fetch.spec.whatwg.org/#request-render-blocking) boolean associated with fetch [request](https://fetch.spec.whatwg.org/#concept-request)


## Potential Spec Changes

Fetch ([whatwg/fetch#1449](https://github.com/whatwg/fetch/pull/1449))
- New boolean field render-blocking added to [fetch timing info](https://fetch.spec.whatwg.org/#fetch-timing-info)
- [Fetching](https://fetch.spec.whatwg.org/#fetching) is modified to set [fetch timing info](https://fetch.spec.whatwg.org/#fetch-timing-info)'s render-blocking field using request's [render-blocking](https://fetch.spec.whatwg.org/#request-render-blocking)

Resource Timing Level 2 ([w3c/resource-timing#327](https://github.com/w3c/resource-timing/pull/327))
- [4.3](https://w3c.github.io/resource-timing/#sec-performanceresourcetiming) : Adding new field to interface : renderBlockingStatus
- A [PerformanceResourceTiming](https://w3c.github.io/resource-timing/#dom-performanceresourcetiming) has an associated  `RenderBlockingStatusType`  render blocking status.
- [4.3.1] Add new `RenderBlockingStatusType` enum which can have 2 defined values. `blocking` if [timing-info](https://w3c.github.io/resource-timing/#dfn-timing-info)'s newly added render-blocking field is true and `non-blocking` if its false
- Getter steps for `renderBlockingStatus` returns `RenderBlockingStatusType` enum

## Security/Privacy Considerations

- The field does not reveal any new cross-origin information. With respect to ResourceTiming, each frame only reports on the resources fetched by itself, and not any child IFRAMEs and hence no new info is made available.

### [Self-Review Questionnaire: Security and Privacy](https://w3ctag.github.io/security-questionnaire/)

> 01.  What information might this feature expose to Web sites or other parties,
>      and for what purposes is that exposure necessary?

It exposes information relating to if a particular resource blocks rendering or not.

> 02.  Do features in your specification expose the minimum amount of information
>      necessary to enable their intended uses?

Yes

> 03.  How do the features in your specification deal with personal information,
>      personally-identifiable information (PII), or information derived from
>      them?

It does not deal with such information.

> 04.  How do the features in your specification deal with sensitive information?

It does not deal with sensitive information.

> 05.  Do the features in your specification introduce new state for an origin
>      that persists across browsing sessions?

No.

> 06.  Do the features in your specification expose information about the
>      underlying platform to origins?

No.

> 07.  Does this specification allow an origin to send data to the underlying
>      platform?

No.

> 08.  Do features in this specification enable access to device sensors?

No.

> 09.  Do features in this specification enable new script execution/loading
>      mechanisms?

No.

> 10.  Do features in this specification allow an origin to access other devices?

No.

> 11.  Do features in this specification allow an origin some measure of control over
>      a user agent's native UI?

No.

> 12.  What temporary identifiers do the features in this specification create or
>      expose to the web?

None.

> 13.  How does this specification distinguish between behavior in first-party and
>      third-party contexts?

No distinction.

> 14.  How do the features in this specification work in the context of a browser’s
>      Private Browsing or Incognito mode?

No difference.

> 15.  Does this specification have both "Security Considerations" and "Privacy
>      Considerations" sections?

No.

> 16.  Do features in your specification enable origins to downgrade default
>      security protections?

No.

> 17.  How does your feature handle non-"fully active" documents?

No difference.

> 18.  What should this questionnaire have asked?

None.

## Changelog
- Update 1 - Spec changes modified to reuse `Request`'s `render-blocking` boolean.
- Update 2 - Added section for Privacy/Security considerations
- Update 3 - Change type to enum as per TAG suggestion
