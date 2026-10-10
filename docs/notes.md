\# Notes



\##Flow

* 12/09: Started inspection of the tables
* 12/09: Ended inspection, deployed the tables in the model without transforming them. Next step I will clean them.
* 13/09: Done transformation of the data.
* 13/09: Date table "Dates" built in DAX, 01/07/2010–30/06/2026 (16 full seasons). Columns: Date, Year, Month, MonthName, SeasonStartYear, Season, TransferWindow.

  * Season runs Jul–Jun (e.g. 2015/16 = 01/07/2015–30/06/2016)
  * Marked as date table on \[Date]
  * Season sorted by SeasonStartYear to avoid alphabetical ordering
  * TransferWindow: Jul–Aug = Summer, Jan = Winter, rest = Other.
  * June left out on purpose — 30 June closes the previous season.
* 13/09: Model relationships done. First measures written: Sales Revenue, Money Spent.
* 13/09: Net Transfer Balance measure added.
* 13/09: Table "Countries" created with just the name of the countries in it.
* 13/09: Created Three measures on that axis: Revenue by Country, Spend by Country, Net Balance by Country.
* 13/09: added "season" calculated column to transfers, using the same Jul–Jun rule as the Dates table and deleted the one that was already on the table by default.
* 14/09: built ClubParents mapping table (DATATABLE, 27 rows) covering Sporting CP, Benfica, FC Porto and SC Braga, main team, B team and all youth entities, mapped club\_id → parent\_id. Main teams map to themselves so every id in the list resolves.
* 14/09: is\_internal\_move column added, using ClubParents. Validated on the Essugo case: the four internal promotions return TRUE, the Chelsea sale and both loans return FALSE.
* 16/09: player\_origin rewritten to ignore internal entries. Now returns Academy correctly.
* 16/09: is\_loan\_out and is\_loan\_return columns added (zero fee, same two clubs, reverse direction, within 550 days). movement\_type rewritten: internal moves and both loan legs fall into "Other", leaving Paid transfer / Free move / Unknown value for actual market moves. player\_origin now ignores internal entries and loan returns.
* 16/09: ROI pairing done with calculated columns on transfers
* 16/09: from\_parent\_id and to\_parent\_id columns added. Base for ROI pairing.
* 16/09: is\_sale column added (Paid transfer + seller's parent is one of the four clubs).
* 16/09: buy\_date and buy\_fee columns added: most recent real entry of the same player into the same parent club before the sale.
* 16/09: buy\_age and sale\_origin columns added. sale\_origin derived from the pairing: no purchase or bought under 18 = Academy, then Unknown / Free / Bought.
* 21/09: Page 2 started: club slicer, sales vs spend by season, Cumulative Net Balance measure.
* 27/09: Added various measures and insights for each club (Benfica, Porto, Sporting and Braga)
* 28/09: Page 1 (Leagues) started.
* 28/09: League Tier calculated column added to Countries: Major leagues (England, Spain, Italy, Germany, France), Secondary leagues (Portugal, Netherlands, Belgium, Turkey, Brazil), Other (everything else).
* 28/09: Hierarchical slicer League Tier → Country added, multi-select without CTRL.
* 28/09: Card for financial information for Portugal added.
* 28/09: Line chart Net Balance by Country by season, legend by country, driven by the slicer.
* 28/09: Scatter Revenue vs Spend by country with symmetry shading as break-even diagonal, both axes starting at 0.
* 28/09: Trade Balance Index measure = net / (revenue + spend), bounded between -100% and +100%. Ranked bar chart, all countries (slicer interaction cut), Portugal highlighted.
* 28/09: Revenue Share measure (country revenue / global revenue), line chart by season.
* 10/10: Page 3 (profile) built, with the club slicer synced with page 2.
* 10/10: "Age at signing vs age at sale": clustered column, Purchase Count and Sales Count by age\_at\_transfer, categorical axis sorted by age.
* 10/10: "Positions bought vs sold": clustered bar by position. Position labels renamed in Power Query (Attack → Forward, Midfield → Midfielder) with Replace Values, match entire cell.
* 10/10: "Nationalities bought vs sold": top 6 nationalities by Player Moves (Sales Count + Purchase Count).
* 10/10: Two donuts: "Where they sell" (sales\_destination by Sales Revenue: England, Spain, France, Italy, Saudi Arabia, Portugal, Other) and "Where they buy from" (purchase\_origin by Money Spent: Portugal, Spain, Brazil, Argentina, Italy, Other).



\## Decisions

* 12/09: Cut in 30/06/2026. Pipeline of the dataset stopped in july of 2026 and there were \~1000 lines with future dates, probably pre-done deals.
* 12/09: Add the dimension "Origin of the player" to the selling club (Bought, Free Agent, Formed)
* 12/09: Going to have European and American leagues to be able to see if, for example, "Portugal makes more money in transfers than England".
* 12/09: From the players dataset I will only use "player\_id", "name", "date\_of\_birth", "position", "sub\_position", "country\_of\_citizenship" (not country of birth since it has 10% more missing values).
* 12/09: Use player\_valuations.current\_club\_id as basis to detect players that were formed and not bought (would be evaluated without transfer)
* 12/09: Formed players will be defined through abstinence, anyway I found teams U23 in the player evaluation dataset, so if needed I can check there.
* 13/09: transformation:

  * players dataset reduced to 6 columns
  * transfers filtered to "transfer\_date" < 01/07/2026
  * competitions dataset filtered to type = "domestic\_league"
* 13/09: Model relationships. clubs links twice to transfers (from\_club\_id active, to\_club\_id inactive), will use  USERELATIONSHIP in DAX to switch between buy side and sell side.
* 14/09: mapping limited to the four clubs on the club page. Extending it to foreign clubs (Ajax, Barcelona) is a nice-to-have for Thursday.
* 16/09: Profit kept nominal as the main metric. Market-adjusted version (median paid fee index by season) planned after nominal ROI is validated.
* 27/09: Loan window widened from 550 to 800 days to cover two-season loans.
* 09/10: Origin-of-sale chart (Academy / Bought / Free over time) moved from page 2 to page 3: it describes the player profile, not the club's business.
* 09/10: buy\_bracket\_order and buy\_bracket columns added to transfers (Under €5m / €5–15m / Over €15m, Bought sales only). Order built from buy\_fee, name built from the order, to avoid the circular dependency hit earlier with Tier Order.
* 09/10: "ROI by purchase price" chart added to page 2: line and clustered column, Weighted ROI as columns, Sales Count as line on a secondary axis, blank bracket filtered out.
* 09/10: New measures: Avg Holding Years (Bought sales only) and Academy Revenue Share (academy share of sale revenue). Both added as cards on page 2.
* 09/10: player\_label rewritten: name + player\_id encoded as invisible zero-width characters (22 bits, UNICHAR 8203/8204). Top 10 shows names only, homonyms stay separate.
* 09/10: Page 2 insights rewritten as a dynamic measure: SWITCH per club for the wording, every number pulled from measures.
* 09/10: Page 3 will reuse the page 2 club slicer, synced across pages.
* 10/10: Page 3 covers who clubs sell and who they buy (buying tastes), using measures instead of a page filter.
* 10/10: Counts for the profile charts (age, position, nationality), money for the leagues (who pays and who gets paid).
* 10/10: Donuts only with an "Other" slice, so percentages refer to the club's full total. Countries chosen from each club's top 6.
* 10/10: Calculated columns instead of Power BI groups for the donut categories: groups keep (Blank) apart, calculated columns send it to Other.



\## Problems

* 12/09: clubs.net\_transfer\_record is a snapshot of the last season (last\_season varies between 2012 and 2025), its in text format and inconsistent.
* 12/09: clubs.total\_market\_value 100% missing
* 12/09: The clubs dataset covers 32 domestic leagues (the 31 competitions from competitions + Colombia). Clubs outside those 32 leagues will appear in transfers but without any information in the clubs dataset.
* 13/09: A blank Season row shows €871m, transfers dated before 01/07/2010, outside the Dates table range. Will be filtered when I get there.
* 13/09: From\_country and to\_country are two different axe, a country sells as origin and buys as destination, so revenue and spend can't sit in one table. Need a disconnected country table to put both on one axis.
* 13/09: Zero academy promotions detected for Portugal (after fixing the patterns). Consistent with clubs having no Benfica B / Porto B: those moves simply aren't in the dataset.
* 14/09: player\_origin returned no "Formed" players, the rule finds an incoming transfer for nearly everyone, including academy graduates (Sporting B → Sporting counts as an entry). Needs the parent-club mapping to tell internal promotions from real acquisitions.
* 14/09: Loans are recorded as ordinary zero-fee transfers, so a player shows repeated entries into the same club (Sporting → Chaves → Sporting). The ROI pairing must find the first real acquisition, not the most recent entry.
* 14/09: Text matching on club names is unusable for Portuguese clubs, "Sporting" alone returns Sporting Macau, Osasco Sporting, TS Sporting, Leader Sporting and others. Going with a manual club\_id → parent\_id mapping table instead.
* 16/09: Club profile biased toward academy players: every academy player eventually leaves, many for small fees, so counting sales inflates the academy share.
* 27/09: Loan detection compares club ids, so B-team loans returning to the first team are not recognised: João Mário and Palhinha get a €0 cost (labelled Free instead of Academy) and holding\_days counted from the loan return (Palhinha 703 days).
* 09/10: First player\_label version failed: "id" is not a valid DAX variable name, then the per-row CALCULATE over the players table ran out of memory. Replaced by the bit-encoding version, which only reads its own row.
* 09/10: Braga's ROI chart showed a 2500% axis: Sales Count shared the axis with ROI (25 sales read as 2500%). Fixed with the secondary y-axis.
* 09/10: Old page 2 insights contained errors: Benfica's "three biggest sales are academy players" was false (Enzo was bought); Benfica "only club with Unknown-cost share" and Braga "largest Unknown-cost share" contradicted each other (both removed); Sporting's model shift belongs on page 3.
* 10/10: Categorical axis sorted by measure value, scrambling ages. Fixed with Sort axis by age\_at\_transfer, ascending.
* 10/10: Grouped donut summed to \~95%: the (Blank) slice stayed out of the Other group. Fixed with the calculated column.
* 10/10: Blank from\_country = clubs not in the clubs table (lower divisions in Portugal and abroad). A sizable share of purchases for Porto, Braga and Sporting; part of "Other" in purchases is Portuguese lower-league clubs.



\## References

* 12/09: Only 17,554 of 99k+ transfers have a fee > 0 (\~18%).
* 12/09: Country Codess: Austria - A1

&#x09;		  Argentina - ARG1

&#x09;		  Australia - AUS1

&#x09;		  Belgium - BE1

&#x09;		  Brazil - BRA1

&#x09;		  Switzerland - C1

&#x09;		  Denmark - DK1

&#x09;		  Spain - ES1

&#x09;		  France - FR1

&#x09;		  England - GB1

&#x09;		  Greece - GR1

&#x09;		  Italy - IT1

&#x09;		  Japan - JAP1

&#x09;		  Croatia - KR1

&#x09;		  Germany - L1

&#x09;		  Mexico - MEX1

&#x09;		  United States - MLS1

&#x09;		  Netherlands - NL1

&#x09;		  Norway - NO1

&#x09;		  Poland - PL1

&#x09;		  Portugal - PO1

&#x09;		  Romania - RO1

&#x09;		  South Korea - RSK1

&#x09;		  Russia - RU1

&#x09;		  Saudi Arabia - SA1

&#x09;		  Scotland - SC1

&#x09;		  Sweden - SE1

&#x09;		  Serbia - SER1

&#x09;		  Türkiye - TR1

&#x09;		  Czech Republic - TS1

&#x09;		  Ukraine - UKR1

* 12/09: Clubs links to competitions through domestic\_competition\_id (PO1, GB1, ...). Path will be club → league → country.
* 12/09: There are more country codes in clubs than in transfers.
* 12/09: The clubs dataset has 35 clubs in PO1, and they are all from the major league, so there is no reference to, for example, Benfica B or Porto B, etc.
* 14/09: Parent club mapping (club\_id → parent):

  * Sporting CP 336: 336, 10949 (B), 67909 (U23), 10283 (U19), 48365 (U17), 53956 (Sub-15), 53957 (Yth.)
  * Benfica 294: 294, 10330 (B), 68780 (U23), 8889 (U19), 45618 (U17), 53954 (U15), 53955 (Yth.)
  * FC Porto 720: 720, 10331 (B), 9844 (U19), 47962 (U17), 56765 (U15), 56766 (Youth)
  * SC Braga 1075: 1075, 10789 (B), 68077 (U23), 12964 (U19), 48758 (U17), 53958 (U15), 53959 (Youth)
* 10/10: Net Transfer vs Total Profit.

  * Net Transfer = all sales revenue, all money spent in the period (2010/11–2025/26). A cash view: every sale and every purchase counted, with no link between them.
  * Total Profit = for each player sold, sale fee - that player's own purchase cost (buy–sell pairing). A per-player view.
  * They differ for two reasons: (1) players bought but never sold (still in the squad, released for free, retired at the club) count as spend in Net Transfer but never enter Total Profit; (2) a player bought before 2010 and sold inside the window enters Total Profit with his original cost, while that purchase falls outside Net Transfer.
  * Example: FC Porto, Total Profit €747m vs Net Transfer €440m. Part of the spending went on players never resold or sold at a loss.
  * In short: Net Transfer = how much cash was left; Total Profit = how much was earned on the players actually sold.



\##Findings

* 13/09: measures validated — global totals for Money Spent and Sales Revenue match exactly (82.9bn), as they must.
* 13/09: Benfica 2010–2026: €793m spent, €1,510m received. Net +€717m.
* 13/09: \~6% of total value sits on a blank club row, transfers whose club isn't in the clubs table (outside the 32 covered leagues).
* 13/09: Global transfer spend grew from €450m in 2010/11 to €10,679m in 2024/25 (\~24x).
* 13/09: sales revenue by selling country, 2010–2026 (top 10): England 14,573m | Italy 10,098m | France 8,808m | Spain 7,806m | Germany 7,635m | Portugal 4,746m | Brazil 3,444m | Netherlands 3,315m | Belgium 2,520m | Argentina 2,171m | Portugal 6th in Europe, ahead of Netherlands and Belgium.
* 13/09: Portugal sells 4,746m, spends 2,290m → net +2,456m. England sells 14,573m, spends 26,328m → net -11,755m. Portugal earns a third of England's transfer revenue but converts it into a large surplus. Netherlands also positive (+1,886m) but at half the margin. Italy roughly break-even (-1,218m).
* 13/09: Saudi Arabia is the mirror image of Portugal, it spends 2,657m, sells 464m, net -2,192m.
* 13/09: Age at transfer distribution: peak at 19-20, tail to 35. Movements from age 5-6 exist, probably academy transfers.
* 13/09: Filtering to transfer\_fee > 0 shifts the age peak from 19-20 to 22-23 and removes almost all the young tail.



* \## Findings
* 13/09: Portuguese academy and B teams DO exist in transfers (Sporting Yth. → Sub-15 → U17 → B → Sporting), they just aren't in the clubs table. Earlier "zero academy promotions in Portugal" was wrong: the filter, not the data, was at fault.
* 13/09: Essugo case also shows loans create repeat entries into the same club (Sporting → Chaves → Sporting → Las Palmas → Sporting). The ROI pairing must find the first real acquisition, not the latest entry. Use this player as the test case.
* 14/09: Sporting academy sales now surface as expected, Cristiano Ronaldo, Nani, João Moutinho, Bruma, Eric Dier all classified as Academy. Strong validation that the rule matches reality.
* 14/09: players who were loaned out still misclassify as Free (Essugo's Chelsea sale), because the return from loan counts as a non-internal entry. Loan detection is the remaining piece.
* &#x20;27/09: Club profile vs margin: Braga \~91% margin with the lowest profit, Benfica \~69% with the highest (€1.0bn), Porto \~72%, Sporting \~75%. Bigger clubs earn more in absolute terms but at a lower margin.
* 27/09: Sporting's sale revenue shifts from mostly Academy (2010-2016) to a large Bought share from 2016/17 onwards.
* 09/10: ROI falls with purchase price at every club with enough sales: Porto 327% (under €5m) vs 46% (over €15m); Benfica 162% vs 48%; Sporting 298% / 100% / 90%.
* 09/10: Braga: 25 of its 27 bought-player sales cost under €5m, none over €15m. Highest academy share (54%), best margin (87%), longest holding (3.7 years), best revenue/spend ratio (2.7x). Its top three deals are all academy players.
* 09/10: Porto: lowest academy share (22%) and best ROI (146%), the buy-and-resell club.
* 09/10: Benfica made 13 purchases over €15m that were later sold, more than the other three combined (9), which drags its ROI down to 69%.
* 09/10: Sporting has the shortest average holding of bought players (2.3 years).
* 10/10: Benfica has an extra purchase peak at 18–20 (prospects bought to develop); sales concentrate at 21–23, with a second rise at 26–27.
* 10/10: Benfica: Portuguese players are mostly sold (37 sales vs 14 purchases); foreign players (Brazil, Argentina, Serbia) are bought and sold in similar numbers.
* 10/10: Braga is the only club whose top sales destination is Portugal (27%), followed by Spain (25%). The other three sell mainly to England.

