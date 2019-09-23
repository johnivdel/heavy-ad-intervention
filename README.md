# Heavy Ad Intervention Explainer
## Background
A small fraction of ads on the web use an egregious amount of system resources. These poorly performant ads (whether intentional or not) harm the user’s browsing experience by making pages slow, draining device battery, and consuming mobile data (for those without unlimited plans).

In these egregious cases, the browser can unload the offending ads to protect the individual’s device resources. This is a strong intervention that is meant to safeguard the user’s resources with low risk because unloading an ad is unlikely to result in loss of functionality of the page’s main content.

## Motivating Use Cases
Examples of observed ad behaviors that are intended to be discouraged:
*   Ads that mine cryptocurrency
*   Ads that load large, poorly compressed images
*   Ads that load large video files before a user gesture
*   Ads that perform expensive operations in javascript, such as decoding video files, or CPU timing attacks

### Non-Goals
*   Discouraging specific ad creative formats, such as display video ads

## Intervention Overview
The user agent will unload ads that use an egregious amount of network bandwidth or CPU usage. We define egregious as using more of a resource than 99.9% of ads as measured by the browser. Only ads that have not been interacted with by the user will be unloaded.

All unloaded frames will be notified via an [Intervention Report](https://github.com/W3C/reporting/blob/master/EXPLAINER.md#basic-report-formats) that the intervention occurred. This feedback is necessary to help advertisers or their ad technology vendors to identify and fix ads that are triggering this intervention.

The classification of ads is left to the discretion of the user agent. For example, Chrome detects ads using its [AdTagging ](https://chromium.googlesource.com/chromium/src/+/master/docs/ad_tagging.md)feature.

### Proposed Thresholds
An advertisement is considered heavy if it has not been clicked on by the user and meets any of the following criteria:
*   Used the main thread for more than 60 seconds total
*   Used the main thread for more than 15 seconds in any 30 second window (50% utilization over 30 seconds)
*   Used more than 4 megabytes of network bandwidth to load resources

The thresholds above were inspired by the IAB’s Lean Standard but chosen by looking at Chrome’s metrics at the 99.9th percentile of network and CPU usage in ads. Threshold numbers were rounded to be readable and memorable. Numbers were picked inclusive of all ads seen by Chrome’s AdTagging, which is better at capturing display ads than native ads.

A total CPU usage threshold is used in addition to a peak usage threshold to discourage ads from using just below the peak window usage, which still drains the user’s battery over time.

Intervening at the above thresholds is projected to save 12.8% of the network usage by ad creatives, and 16.1% of all CPU usage by ad creatives, with most of that value going to individual users.

Thresholds are agnostic to platform (desktop / mobile) so that creative authors can easily know if their ad would be subject to intervention or not. Thresholds may need to change as the web ecosystem and common device profiles evolve. 

## Privacy
Unloading an ad is easily observed by a web page. This poses some privacy concerns with respect to the [same-origin policy](https://developer.mozilla.org/en-US/docs/Web/Security/Same-origin_policy). For example, a malicious actor could create an ad, load an x-origin resource/frame, and observe whether it was unloaded. This leaks information about the size and CPU usage of this x-origin resource/frame.

User agents can mitigate this by adding random fuzz to the thresholds and throttling interventions on origins that have high per-user incident rates. Our strawman proposal is 1MB of random fuzz for the network threshold, and 5 interventions per top frame origin per user per day. 
