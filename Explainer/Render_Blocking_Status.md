# Render Blocking Status

Proposal to add a new field `render blocking status` to PerformanceResourceTiming which holds a string highlighting the status of stylesheets and scripts.

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
    readonly attribute DOMString render_blocking_status;
    ...
    ...
    [Default] object toJSON();
};
```

Sample usage:
```javascript
const entry_list = performance.getEntriesByType("resource");
console.log(entry_list[0].render_blocking_status);
```


## Render Blocking Status Values

The following values are proposed for the resource blocking status

`blocking` - A potentially render blocking resource

`non-blocking` - Not blocking resource

We primarily reuse the heurestics used to set [render blocking](https://fetch.spec.whatwg.org/#request-render-blocking) boolean associated with fetch [request](https://fetch.spec.whatwg.org/#concept-request)


## Potential Spec Changes

Resource Timing Level 2 ([w3c/resource-timing#327](https://github.com/w3c/resource-timing/pull/327))
- [4.3](https://w3c.github.io/resource-timing/#sec-performanceresourcetiming) : Adding new field to interface : renderBlockingStatus
- [4.7](https://w3c.github.io/resource-timing/#marking-resource-timing) : [Mark resource timing](https://w3c.github.io/resource-timing/#dfn-mark-resource-timing) gets a renderBlockingStatus string and [setup resource timing entry](https://w3c.github.io/resource-timing/#dfn-setup-the-resource-timing-entry) uses it while creating the entry


Fetch ([whatwg/fetch#1449](https://github.com/whatwg/fetch/pull/1449))
- [Finalize and report timing](https://fetch.spec.whatwg.org/#finalize-and-report-timing) is modified to accept a render blocking status string (defaults to "unset") and passes the same to mark resource timing (step 11)
    
    
HTML ([whatwg/html#7979](https://github.com/whatwg/html/pull/7979))
- `<link>` element 
    - Modify [default-fetch-and-process-the-linked-resource](https://html.spec.whatwg.org/#default-fetch-and-process-the-linked-resource) to pass in the render blocking status based on render blocking boolean from request to finalize-and-report-timing (in Step 6.1)
    - Pass in value as "non-blocking" to finalize in Step 14 of [preload](https://html.spec.whatwg.org/multipage/links.html#preload)
- `<script>` element
    - For the algorithms <a href="https://html.spec.whatwg.org/#fetch-a-classic-script">fetch-a-classic-script</a> and <a href="https://html.spec.whatwg.org/#fetch-a-single-module-script">fetch-a-single-module-script</a> modify the step with ‘Finalize and report timing’ to pass in a render blocking status based on render blocking boolean from script-fetch-options.
