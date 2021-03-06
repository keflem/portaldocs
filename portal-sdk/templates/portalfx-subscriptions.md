<properties title="" pageTitle="Working with subscriptions" description="" authors="flanakin" />

## Subscriptions

Extensions need subscription details to enable the following scenarios:

* Determining if a resource is in a disabled subscription
* Showing the subscription display name in resource properties
* Making a decision based on the quota id
* Showing resources for all filtered subscriptions

### Getting subscription details
Most extensions should only need the details of a single subscription for these first 3 scenarios. To get subscription details, call the `getSubscriptionInfo()` function:

```ts
MsPortalFx.Azure.getSubscriptionInfo(subId).then((sub: Subscription) => {
    var name = sub.displayName,
        isDisabled = sub.state.toLowerCase() === "disabled",
        quotaId = sub.subscriptionPolicies.quotaId;
    ...
});
```

### Browse
Extensions that have subscription resources should use [Browse v2](/documentation/articles/portalfx-browse#resources), which handles the complexity of aggregating resources across subscriptions on behalf of the extension. All supplemental data should be obtained using the specified resource ids to ensure the extension isn't querying or polling for more data than is actually necessary.


### Subscription filtering on blades and parts
If you show content across subscriptions, use the following guidelines:

1. Locked blades should add the `ResourceFilterPart` to the top of their blade (see below)
2. Unlocked blades should add a subtitle that represents the selected subscriptions and a `Filter` command that includes the subscription filter in a context pane (same as locked blades)
3. Parts should add a subtitle that represents the selected subscriptions and expose a `Filter` command similar to unlocked blades

The selected subscription label should be formatted using these rules:

* 1 subscription: "{subscription.displayName}"
* 2+ subscriptions: "{count} subscriptions"
* All subscriptions: "All subscriptions"

> [WACOM.NOTE] We will provide an API to do the label calculation for you. Stay tuned...

If you have a locked blade that shows resources across subscriptions, you can add the `ResourceFilterPart` from Hubs and bind its outputs to your part's inputs:

```xml
<PartReference Name="Filter" PartType="ResourceFilterPart" InitialSize="FullWidthFitHeight">
  <PartReference.PropertyBindings>
    <Binding Property="textFilterEnabled" Source="{Constant true}" />
  </PartReference.PropertyBindings>
</PartReference>
<PartReference Name="List" PartType="BrowseResourceListPart" InitialSize="FullWidthFitHeight">
  <PartReference.PropertyBindings>
    <Binding Property="filter"
             Source="{PartProperty Part=Filter, Property=filter}" />
    <Binding Property="selectedSubscriptions"
             Source="{PartProperty Part=Filter, Property=subscriptions}" />
    <Binding Property="subscriptionsFiltered"
             Source="{PartProperty Part=Filter, Property=subscriptionsFiltered}" />
  </PartReference.PropertyBindings>
</PartReference>
```

The following optional part inputs are supported:

| Input property             | Values |
|----------------------------|--------|
| textFilterEnabled          | Optional. Specifies whether the text filter is visible. Defaults to false. |
| subscriptionsFilterEnabled | Optional. Specifies whether the subscription filter is visible. Defaults to true. |
| textFilterPlaceholder      | Optional. Specifies the text filter placeholder when empty. Defaults to "Filter items..." |
| selectedSubscriptionId     | Optional. Specifies the subscription id to show. This will hide the subscription filter. Defaults to null. |

Bind the following outputs to your part:

| Output property       | Values |
|-----------------------|--------|
| filter                | Text filter value |
| selectedSubscriptions | Array of selected subscriptions |
| subscriptionsFiltered | Boolean value to indicate whether subscriptions have been filtered or not |

