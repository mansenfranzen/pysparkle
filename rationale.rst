=========
Rationale
=========

Spark is a state-of-the-art distributed computation engine. It scales
from single local machine to clusters of thousand nodes. It offers both a
SQL and DataFrameAPI interface to translate business requirements into a logical
representation. It has bindings for Python, Java and R.

Spark's DataFrameAPI vs. cloud native SQL Interfaces
====================================================

This article is intended for data analysts, scientists, engineers and related professions such as solution architects who deal with distributed computation engines in their day to day work. It is an opiniated comparison between Spark's DataFrameAPI and cloud native SQL interfaces (e.g. Presto/Athena, Snowflake, Redshift and alike). It does not reiterate on the shortcomings of the SQL language itself which has already been extensively discussed by others (https://www.edgedb.com/blog/we-can-do-better-than-sql). Rather, it outlines benefits of Spark's DataFrameAPI in contrast to seemingly more easier to use text based SQL. By the end of the article, you will be challenged to think about why Spark's DataFrameAPI and native SQL interfaces are complementary and where each has its justification.

DataFrameAPI and SQL
--------------------

Let's first focus on the relevant distinction between the DataFrameAPI and SQL interface. While both are used to express data processing logic, they differ in the way they formulate ETL/ELT pipelines.

SQL is probably the oldest language with the most widespread adoption in recent years due to the rise of big data. It was originally invented in the 1970s for managing and processing relational data. Back then, nobody could have guessed its todays popularity. Being half a century old, it might not be a surprise that SQL exposes a purely string based interface. While ORMs and related frameworks have developed over time as higher level abstractions to generate SQL, it is still mostly written in plain text by most practioners in production environments today. 

In contrast, Spark's DataFrameAPI is embedded in general purpose languages like Python or Java. Interestingly, these languages first came into existance roughly 20 years later than SQL. While SQL formulates concepts like tables, projections, filters and transformations as pure text, the DataFrameAPI maps these into classes. More concretely, entities like tables (aka dataframes) gain an explict object representation with attributes (e.g. column names and dtypes) and methods (eg. filter, join and aggregate). Hence, it is called a DataFrame *Application Programming Interface* (API) because it allows to directly interact with a dataframe object representation.

Next, let's focus on the implications that can be derived from the distinction of a plain text interface versus a programming language interface.  

Managing complexity
-------------------

Data pipelines often inherit complexity from business demands like time series analysis. This typically inludes multiple joins, aggregations, sophisticated window functions and more. Handling such complexity is a difficult task and asks for simplification via appropriate abstractions.

Since a DataFrameAPI is embedded in a programming language, it allows to leverage software engineering concepts such as inheritance, composition, introspection and higher order functions. This provides data engineers a toolbox of abstractions to properly handle complexity. More conretely, it allows to structure, reuse and dynamically generate building blocks of data pipelines.

In contrast, the standard SQL interface is not embedded in a programming language and remains string based. Therefore, the before mentioned programming techniques can't be applied. Due to the lack of abstraction, SQL statements often tend to result in complex, cluttered and hard to reason strings. 

Frameworks like dbt, matillion and others address this shortcomings by delegating SQL string generation to function like abstractions. However, they do not solve the core problem of missing a proper object representation as they do not have an inherent understanding of what objects they deal with.

Testing
-------

Since Spark's DataFrameAPI lives within the wider ecosystem of a programming language like python, rich testing frameworks such as pytest can be leveraged for great versatility to:

- dynamically generate tests via parametrization
- combine and manage tests into categories via marks
- run only relevant tests via flexible filters
- provide meaningful differences in failed tests
- centrally collect test results and rerun failed tests

Accordingly, data engineers can develop unit tests on individual transformations or integration tests on entire piplelines to ensure semantic correctness of the data pipeline (also known as logic tests). Moreover, advanced techniques like property-based testing or mutation testing can be employed to automatically reveal edge cases. Once the test suite is set up, refactorings can be safely done. 

Drastically, there is no dedicated testing framework in the SQL space. This is actually not specific to cloud based SQL engines but is valid for SQL databases in general. Dbt attempts to solve some of these issues while providing a minimal testing framework. However, this is in no sense comparable with a dedicated testing framework like pytest. Moreover, dbt tests mainly focus on shape, integrity and related data quality measures (also known as data tests). It can be further extented with ``great expectations`` which offers a huge suite of additional tests. However, it still fails to test the semantic correctness of a data pipeline.

In addition, spark can be run locally in standalone mode on a developer's machine or in a continuous integration (CI) environment. In contrast, cloud native SQL engines cannot be run locally or in a CI environment without cloud access. Each time you test, you need to spin up cloud compute. Each time you test, you will likely need to create lots of temporary input and output tables. Cloud SQL testing is both slower and likely more expensive than local or CI based testing.

Debugging
---------

With increasing complexity, errors and incorrect results will eventually occur sooner or later. Finding and solving bugs in a Spark data pipeline follows the same principle as in traditional software development. Using a debugger of one's favourite IDE allows to step through every line of your code while being able to interactively follow and modify the state of the data pipeline for full observability. Even though Spark's execution model is lazy, you can always use a ``show`` or ``toPandas`` on intermediate dataframe representations to inspect results.

This is not possible with SQL. For Spark's DataFrameAPI, the smallest observable unit is a line of code or dataframe statement. For SQL, the smallest observable unit is a complete view or table. Accordingly, to properly debug in SQL, one needs to create lots of intermediate views or tables which is both a manual and tedious process.


Declarative and imperative paradigm
-----------------------------------

SQL is declarative by heart. The user describes *what* result is desired. The underlying engine determines *how* this result needs to computed with best performance in mind.

This is well suited in more than 95% of the cases for two reasons. First, the typical end user only cares about the result. Second, vendors of SQL computation
engines have decades of experience in optimizing queries and generating the best physical execution plans.

However in the remaining 5%, especially with increasing complexity, even well optimized engines will eventually fail to find the best execution plan. This is especially true in a distributed environment where the correct partitioning and physical layout of the data is key. If not handled correctly, data skew and inappropriate parallelism might result in insufficient memory (e.g. disk spills or even out of memory exception) and low CPU utilization.

Spark's DataFrameAPI provides the possibility to imperatively adjust the computation graph. More concretely, Spark's DataFrameAPI allows to dictate *how* the computation
plan is generated on a lower level. For example, the partitioning of the data can be manually enforced via ``repartition``. Moreover, intermediate results
can be explicitly stored via ``cache`` to prevent expensive re-computations. Last but not least, join strategies may be directly provided via join hints (e.g. broadcast joins).

Required skill and ease of use
------------------------------

The above mentioned advantages of Spark's DataFrameAPI do not come for free. In contrast to SQL, Spark's DataFrameAPI requires proficiency in a programming language. Setting up tests for spark on a local machine and in CI can be difficult at first. Moreover, Spark forces its users to explicitly think about its distributed computation and lazy execution model to effectively make use of parellism and caching. This is hidden and abstracted away in SQL. Last but not least, setting up a Spark cluster is still more complicated and requires more thought than using native SQL compute backends even though this has greatly improved via managed services such as AWS Glue and Databricks.

Best of both worlds
-------------------

While SQL is the lingua franca for data analysis, it might not be the best choice for everything. SQL is great for dashboarding and BI use cases with simple queries for which the DataFrameAPI is rather over-engineered. However, business critical data pipelines with high complexity and volume are better suited to be implemented via a DataFrameAPI interface because:

- complexity can be handled with well established concepts from software engineering
- semantic correctness can be guaranteed with dedicated testing frameworks
- developer productivity can be greater due to automation and debugging superiority
- pipeline execution can be better profiled and optimized

Hence, SQL and DataFrameAPI interfaces are complementary with each having its strengths and justification.

Motivation
==========

While pyspark offers great flexibility
