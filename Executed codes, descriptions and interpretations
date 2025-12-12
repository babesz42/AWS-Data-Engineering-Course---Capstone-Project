# Executed codes, explonations and interpretations
I will now interpret the queries I have ran in Athena. The codes that I have ran in the Cloud9 IDE are written in the other file and those are also pretty self explanatory.

# Query 1
```
SELECT year, fishing_entity AS Country, CAST(CAST(SUM(landed_value) AS DOUBLE) AS DECIMAL(38,2)) AS ValueAllHighSeasCatch
FROM fishdb.data_source_6769
WHERE area_name IS NULL and fishing_entity='Fiji' AND year > 2000
GROUP BY year, fishing_entity
ORDER By year
```
