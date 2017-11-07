Merge analysis
================
Ian D. Gow
07/11/2017

Proxy Insight
-------------

For this I first performed an inner join of the `proposals`, `meetings`, and `issuer` tables, which yielded 1,652,013 unique proposals (by `proposal_id`) spread out over 31,639 companies (by `pid`).

I then pulled in the `synth_pva` table, which contains synthetic recommendations from both ISS and GL and found an overlap with the above table of 396,436 unique proposals (by `proposal_id`) spread out over 10,783 companies (by `pid`).

unique proposals (by `proposal_id`) spread out over companies (by `pid`).

If you filter this by `country_name=="US"`, there is just coverage for 148,623 proposals spread over 4,267 companies. This means proposals where there is a GL rec, an ISS rec, or both.

WRDS ISS
--------

For this I performed an inner join of the `risk.issrec` and `risk.vavoteresults` tables, which yielded 475,586 unique proposals (by `meetingdate` and `itemonagendaid` grouped) spread out over 10,213 companies (by `companyid`).

In this cut, there are 251 unique proposals where `issrec` was either Null or "None"", leaving 475,335 unique proposals with ISS recommendations.

If you filter this by `countryofinc=="USA"`, there is coverage for 459,224 proposals spread over 9,985 companies.

Merger
------

I then attempted to merge the PI and ISS datasets together to see how many overlapped. For this, I considered just US companies.

I started with 318,273 proposals from Proxy Insight (US only). Note that this includes proposals where there is neither a GL rec nor an ISS rec.

I also started with 459,224 proposals from WRDS ISS (US only). This also includes proposals where there was no ISS rec.

However, when I considered which variables to use to merge the two datasets together, I ran into some issues. The combination of variables that I decided to join on were:

-   **Proxy Insight:** `(cusip, meeting_date, proposal_number_orig, proposal_order_row)`
-   **WRDS ISS:** `(cusip, meetingdate, ballotitemnumber, seqnumber)`

Looking closer, though, I found that:

-   **Proxy Insight**: 318,273 proposals when looking at "proposal\_id", but 315,316 proposals when grouping on the four variables above.
-   **WRDS ISS**: 459,224 proposals when looking at "meetingdate"+"itemonagendaid" combination, but 458,121 proposals when grouping on the four variables above In other words, there appears to be duplicate items in each dataset based on those four variables and I am not sure what other common variable to use to distinguish the duplicates.

Despite the duplication, I performed a join of the two datasets on the four variables anyway.

Here I elected to keep all of the Proxy Insight proposals since we will need the `proposal_id` to match up with the investor data later. **Note:** *Not sure what this means.*

The resulting merged table has 225,781 rows where there was a match with WRDS ISS. **Note:** *I get 318,729 rows. Not sure what I'm doing that's different.*

``` r
pi_merged_us %>% 
    mutate(year = date_part('year', meeting_date)) %>% 
    count(year) %>% 
    arrange(year) %>% 
    kable()
```

|  year|      n|
|-----:|------:|
|  2007|      7|
|  2008|   2995|
|  2009|  21854|
|  2010|  24061|
|  2011|  30960|
|  2012|  38472|
|  2013|  42012|
|  2014|  40140|
|  2015|  39665|
|  2016|  39546|
|  2017|  38561|

``` r
iss_merged_us %>% 
    mutate(year = date_part('year', meetingdate)) %>% 
    count(year) %>% 
    arrange(year) %>% 
    kable()
```

|  year|      n|
|-----:|------:|
|  2003|  18598|
|  2004|  19808|
|  2005|  19365|
|  2006|  21087|
|  2007|  21389|
|  2008|  22348|
|  2009|  24199|
|  2010|  23601|
|  2011|  34402|
|  2012|  35881|
|  2013|  45955|
|  2014|  46482|
|  2015|  45579|
|  2016|  44280|
|  2017|  36298|

**Note:** *I did not go past here.*

However, 904 of the rows are affected by duplicated `proposal_id` values (comprising 448 `proposal_id` values). For 146 of these duplicated `proposal_id` rows, it looks like the ISS data is *exactly* the same except for differences in the "meetingid" and "itemonagendaid" variables. This accounts for 73 proposals.

There are another 565 rows where one of the duplicated "proposal\_id" rows has the PI-provided "proposal\_text" exactly equal to the WRDS ISS provided "itemdesc" so it is safe to assume that the match is good. The other duplicated rows appear to have very different values for those two columns. This accounts for 281 proposals. The remaining 193 rows are less clean to deal with. There are rows where the "proposal\_text" and "itemdesc" values are very close, but off by one or two letters/words. These can just be evaluated by hand. **Note:** *I don't we want to do anything by "hand", except with some care and with some kind of audit process. This may require more effort than we want to invest.*
