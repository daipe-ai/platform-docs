# Developing Features old way

This is roughly the standard way how features are written.

```python
df = spark.read.table("silver.tbl_loans")

personal_features_df = (
    df.select("UserName", "Age", "Gender", "WorkExperience")
    .groupBy("UserName")
    .agg(
        f.max("Age").alias("Age"),
        f.first("Gender").alias("Gender"),
        f.first("WorkExperience").alias("WorkExperience"),
    )
    .withColumn("run_date", f.lit(today))
)

personal_features_df.write.format("delta").save("...")
```

This approach has many disadvantages

- You have to handle writing the dataframe with features by your self
- Every developer might write features to different locations
- Might be slow or running in conflicts when multiple notebooks try to write features in same location
- None features metadata information
- No track of what was computed and when

Feature store tries to address those problems.
