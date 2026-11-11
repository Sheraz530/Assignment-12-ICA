Court verdicts, cleaning log and reflection

I described the file to an AI tool in words only (columns, types and the kinds of problems I found, no data pasted) and asked for cleaning code. It gave me eight lines. I ran them on a copy and judged each one. The code is in 2_work_code.txt, cell 3 and cell 4.

Line A, item title case. Verdict: modify. It does turn "oil b" into "Oil B", but that is the wrong fix. After running it, Oil B City 1 on 5 Feb has two rows, one at 108.5 and one at 318. The 108.5 is really a Staple A price. I replaced it with a targeted relabel of that one row, flagged, with the old label kept.

Line B, replace the comma in price. Verdict: keep. It turns "1,250" into 1250.0 and nothing else changes. But it only parses the number, it does not make the number right, so the 1250 still needs a decision, which I made separately.

Line C, to_numeric with errors coerce. Verdict: modify. Coerce silently turns anything unreadable into NaN, so I added a check. Missing prices were 1 before and 1 after, so it did not hide anything new this time. I would not trust it without that print.

Line D, to_datetime. Verdict: keep. No failures, 12 unique dates, all 7 days apart.

Line E, drop_duplicates. Verdict: keep. Rows went from 97 to 96 and the one removed was the last row of the file, identical to the 8 Jan Staple A City 1 row. No valid observation was lost.

Line F, keep only price under 1000. Verdict: reject. I expected it to remove 2 rows, the 9999 and the 1250. It removed 3. The third is the blank price on 1 Jan, because NaN is not less than 1000, so it is quietly thrown out. Rows go from 96 to 93 with no message. It also uses a fixed cutoff that I did not choose, and it deletes whole rows when only the price was bad.

Line G, fill missing price with the median. Verdict: reject. After line F there are no missing prices left (I printed it, 0), so this line never does anything. Anyone reading the code would think the blank was filled, when it was actually deleted a line earlier. And if it had run, the median of the whole column is 200, a number mixing oil, sugar and staples. It happens to be near Pulse C, but that is luck, and a filled number would look like a real observation.

Line H, drop unit and data_label. Verdict: reject. data_label is the only place saying this is illustrative teaching data and not real PBS prices. Without it the table looks real. I kept both columns.

So that is 3 kept as they were (B, D, E), 2 modified (A and C), and 3 rejected (F, G, H). A is the one I ended up replacing with my own approach.

Cleaning log. These are all the changes I made to the data.

1 Jan, Pulse C, City 2: price was blank. Kept the row, price stays missing, flag says blank in source.

15 Jan, Staple A, City 2: price was "1,250". Parsed to 1250, then set to missing because it is over ten times the neighbours and I cannot know the real value. Original text kept in price_raw, flagged.

22 Jan, Sweetener D, City 1: price was 9999. Looks like a placeholder code. Set to missing, flagged. Original kept in price_raw.

5 Feb, City 1, labelled oil b with price 108.5: relabelled Staple A. Reasons: the price fits Staple A, Oil B City 1 already has 318 that week, Staple A City 1 has no row otherwise, and Staple A City 2 is 114.5 (6 more). Flagged as my inference. Original label kept in item_raw. This one should be confirmed with whoever entered the data.

Last row of the file, 8 Jan, Staple A, City 1, 99.5: exact duplicate of an earlier row, removed.

Date column changed from text to real dates. Price column changed from text to numbers. unit and data_label left exactly as they were.

After cleaning I have 96 rows, every date, item and city combination appears exactly once, and there are 3 missing prices, which are the 3 I flagged on purpose. As a last sanity check, City 2 minus City 1 is exactly 6.0 in all 45 pairs where both exist, including the relabelled row, which supports the relabel.

Reflection

My decisions differed from the AI's in one main way. The AI deleted or filled things so the table looks tidy. I kept every row and marked the bad values instead, so the table stays honest about what is unknown. Mine is better supported by the checks. My version has 96 rows with every key exactly once. The AI version has 93 rows, one key (Oil B, City 1, 5 Feb) that shows up twice, and 4 expected keys that are gone completely. That is the 3 bad price rows plus the real Staple A City 1 row from 5 Feb, which the AI never had because it was sitting there as Oil B.

What worries me most is that a summary would not have caught this. The overall mean price is 203.61 in both versions. You only see the difference by item: Oil B has 25 observations in the AI version instead of 24, and its mean drops from 324.83 to 316.18 because of the stray 108.5.

What I would have missed alone is probably the NaN less than 1000 thing. I would have read line F as only removing the two big values and never noticed the blank row going with them, or that line G then does nothing. I only saw it once I printed the rows F removed. What the AI missed was the mislabelled row, it treated oil b as a spelling problem.

Things I am less sure about. The relabel to Staple A is an inference, a strong one, but still a guess. Setting 1,250 to missing is cautious, it could be a typo for something like 125, but I cannot prove which. Both are flagged so someone with the source can overrule me.
