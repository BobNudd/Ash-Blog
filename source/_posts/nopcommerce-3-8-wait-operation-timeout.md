---
title: NopCommerce 3.8 - Wait Operation Timeout
date: 2018/06/17 13:48:00
tags: nopcommerce
---

With NopCommerce I’ve noticed  high waits on sql querys resolving when hitting a database in a test environment for local development. This problem becomes more apparent when your seed / test dataset is quite large.

When logging into the Administration area, I would frequently get the following, despite setting a high connection timeout period:

{% codeblock %}
Exception Details: System.ComponentModel.Win32Exception: The wait operation timed out
An unhandled exception was generated during the execution of the current web request. Information regarding the origin and location of the exception can be identified using the exception stack trace below.
Stack Trace:
[Win32Exception (0x80004005): The wait operation timed out]
The stack trace pointed towards PageList.cs class, the method hit returns an IQueryable DbSet. The initial Administrator dashboard performs a lot of I/O reads which
has a large contribution towards the timeout issue.
{% endcodeblock  %}

### Solution

All the dashboard statistic calls are within a single View which can be found at:

`Nop.Admin/Views/Home/Index.cshtml`

To avoid running these on your machine, use the `HttpContext.Current.IsDebuggingEnabled` and wrap in an `If`, like below:

{% codeblock lang:html %}
<div class="content">
    <div class="row">
        <div class="col-md-12">

            @if (!HttpContext.Current.IsDebuggingEnabled)
            {
                @Html.Widget("admin_dashboard_top")
                if (!Model.IsLoggedInAsVendor)
                {
                    <div class="row">
                        <div class="col-md-12">
                            @Html.Action("NopCommerceNews", "Home")
                        </div>
                    </div>
                }
                @Html.Widget("admin_dashboard_news_after")
                if (!Model.IsLoggedInAsVendor && canManageOrders && canManageCustomers && canManageProducts && canManageReturnRequests)
                {
                    <div class="row">
                        <div class="col-md-12">
                            @Html.Action("CommonStatistics", "Home")
                        </div>
                    </div>
                }
                @Html.Widget("admin_dashboard_commonstatistics_after")
                if (!Model.IsLoggedInAsVendor && (canManageOrders || canManageCustomers))
                {
                    <div class="row">
                        @if (!Model.IsLoggedInAsVendor && canManageOrders)
                        {
                            <div class="col-md-6">
                                @Html.Action("OrderStatistics", "Order")
                            </div>
                        }
                        @if (!Model.IsLoggedInAsVendor && canManageCustomers)
                        {
                            <div class="col-md-6">
                                @Html.Action("CustomerStatistics", "Customer")
                            </div>
                        }
                    </div>
                }
                @Html.Widget("admin_dashboard_customerordercharts_after")
                if (!Model.IsLoggedInAsVendor && canManageOrders)
                {
                    <div class="row">
                        <div class="col-md-8">
                            @Html.Action("OrderAverageReport", "Order")
                        </div>
                        <div class="col-md-4">
                            @Html.Action("OrderIncompleteReport", "Order")
                        </div>
                    </div>
                }
                @Html.Widget("admin_dashboard_orderreports_after")
                if (!Model.IsLoggedInAsVendor && (canManageOrders || canManageProducts))
                {
                    <div class="row">
                        @if (canManageOrders)
                        {
                            <div class="col-md-8">
                                @Html.Action("LatestOrders", "Order")
                            </div>
                        }
                        <div class="col-md-4">
                            @if (canManageProducts)
                            {
                                @Html.Action("PopularSearchTermsReport", "Common")
                            }
                        </div>
                    </div>
                }
                @Html.Widget("admin_dashboard_latestorders_searchterms_after")
                if (canManageOrders)
                {
                    <div class="row">
                        <div class="col-md-6">
                            @Html.Action("BestsellersBriefReportByQuantity", "Order")
                        </div>
                        <div class="col-md-6">
                            @Html.Action("BestsellersBriefReportByAmount", "Order")
                        </div>
                    </div>
                }
                @Html.Widget("admin_dashboard_bottom")
            }


        </div>
    </div>
</div>
{% endcodeblock %}

Depending on the context and environment of the project, you may want to write a more robust solution, however for local debugging this is perfectly fine.