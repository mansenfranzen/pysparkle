=========
Rationale
=========

Spark is a state-of-the-art distributed computation engine. It scales
from single local machine to clusters of thousand nodes. It offers both a
SQL and DataFrameAPI interface to translate business requirements into a logical
representation. It has bindings for Python, Java and R.

Spark's DataFrameAPI vs. (cloud) native SQL Interfaces
======================================================

This article is intended for data analysts, scientists, engineers and related professions such as solution architects. It is an opiniated comparison between Spark's DataFrameAPI and cloud native SQL interfaces (e.g. Snowflake, Redshift, Athena and alike). It aims to outline benefits of Spark's DataFrameAPI in contrast to seemingly more easier to use cloud native SQL engines. By the end of the article, you will be challenged to think about why Spark's DataFrameAPI and native SQL interfaces are complementary and where each shines most.

DataFrameAPI and SQL
--------------------

Let's first outline the relevant distinction between the DataFrameAPI and SQL interface. While both are used to express data processing logic, they differ in the way they formulate ETL/ELT task.

SQL is probably one of the oldest languages with most widespread adoption in recent years due to the rise of big data in the 2020s. It was originally invented in the 1970s for managing and processing relational data. Back then, nobody could have guessed it todays popularity. Being half a century old, it might not be a surprise that SQL exposes a purely string based interface. 

In contrast, Spark's DataFrameAPI is embedded in general purpose languages like python or java. Interestingly, these languages first came into existance roughly 20 years later than SQL. While SQL formulates concepts like tables, projections, filters and transformations as pure text, the DataFrameAPI maps these into objects within the embedding programming language. More concretely, entities like tables (aka dataframes) gain an explict object representation with attributes (e.g. column names and dtypes) and methods (eg. filter, join and aggregate). Hence, it is called a DataFrame *Application Programming Interface* (API) because it allows to directly interact with a dataframe while leveraging the full functionality and flexibility of the embedding programming language. 

Having this distinction between DataFrameAPI and SQL interfaces in mind, let's focus on the implications of it. 


Managing complexity
-------------------

Data pipelines often inherit complexity from demands like time series analysis. This typically inludes multiple joins, groupby-aggregates, sophisticated window functions and more. Handling such complexity is a difficult task and asks for simplification via appropriate abstractions.

Since DataFrameAPI is embedded in a programming language, it allows to leverage software engineering concepts such as inheritance, composition, introspection and higher order functions. More concretely, this enables data engineers to structure, reuse and dynamically generate building blocks of data pipelines. These techniques can be seen as a toolbox to create abstractions to handle complexity in transformation demands.

In contrast, an SQL-only interface is not embedded in a programming language and remains string based. Therefore, the before mentioned programming techniques can't be applied. Due to the lack of abstraction, SQL statements often tend to result in complex, cluttered and hard to reason strings. 

Frameworks like dbt, matillion and others address this shortcomings by delegating SQL string generation to function like abstractions. However, they do not solve the core problem of missing a proper object representatio as they do not have an inherent understanding of what objects they deal with.

Testing
-------

Since Spark's DataFrameAPI is embedded in a programming language, rich testing frameworks like pytest can be leveraged for great versatility to:

- dynamically generate tests via parametrization
- combine and manage tests into categories via marks
- run only relevant tests via flexible filters
- provide meaningful differences in failed tests
- centrally collect test results and rerun failed tests

Accordingly, data engineers can develop unit tests on individual transformations or integration tests on entire piplelines to ensure semantic correctness of the data pipeline. Moreover more sophisticated techniques like property-based or mutation testing help to cover edge cases. Once the test suite is set up, refactorings can be safely done. 

In addition, spark can be run locally in a standalone mode on a developer's machine or in a continuous integration (CI) environment. Taken together, this allows software engineering best practices to be applied on the development of data pipelines.

In contrast, there does not seem to be a competitive equivalent in the cloud native sql space. Many SQL engines cannot be run locally or in a CI environment without cloud access. Each time you test, you need to spin up cloud compute. Each time you test, you will likely need to create lots of temporary input and output tables. Cloud SQL testing is both slower and likely more expensive than local or CI based testing.

More drastically, there is no dedicated testing framework in the sql space (this is not specific to cloud based sql engines but is valid for sql engines in general). Dbt attempts to solve some of these issues while providing a minimal testing framework. However, this is in no sense comparable with a dedicated testing framework like pytest. Moreover, dbt tests mainly focus on shape, integrity and related data quality measures (e.g. not null). It can be further extented with ``great expectations`` which offers a huge suite of additional tests. However, it still fails to test the semantic correctness of a data pipeline.

Debugging
---------

Finding and solving bugs in a spark data pipeline follows the same principle as in traditional software development. Using a debugger within your favourite IDE allows you to step through every line of your code while being able to interactively follow and modify the state of the data pipeline for full observability. Even though Spark's execution model is lazy, you can always use a ``show`` or ``toPandas`` on intermediate dataframe representations to inspect results.

This is not possible with SQL. For Spark's DataFrameAPI, the smallest observable unit is a line of code or dataframe statement. For SQL, the smallest observable unit is a complete view or table. Accordingly, to properly debug in SQL, one needs to create lots of intermediate views or tables which is both a manual and tedious process.


Declarative and imperative paradigm
-----------------------------------

SQL is declarative by heart. The user describes *what* result is required. The underlying engine determines *how* this result needs to computed for best performance.

This is well suited in more than 95% of the cases for two reasons. First, the
typical end user only cares about the result. Second, vendors of SQL computation
engines have decades of experience in optimizing queries and generating the best
physical execution plans.

However in the remaining 5%, especially with increasing complexity, even well
optimized engines will eventually fail to find the best execution plan. This is especially true in a
distributed environment where the correct partitioning of the data is key. If
not handled correctly, data skew and inappropriate parallelism might result in
insufficient memory (with disk spills or even out of memory exception) and low
CPU utilization.

Spark's DataFrameAPI provides the possibility to imperatively adjust the
computation graph. More concretely, DataFrameAPI allows to dictate *how* the computation
plan is generated on a lower level. For example, the partitioning of the data
can be manually enforced via ``repartition``. Moreover, intermediate results
can be explicitly stored via ``cache`` to prevent expensive re-computations.
Last but not least, join strategies may directly provided via join hints
(e.g. broadcast joins).

Required skill
--------------

The advantages of Spark's DataFrameAPI do not come for free. In contrast to SQL, Spark demands for more software development abilites. For example, proficiency in a programming language is mandatory. Setting up tests for spark on a local machine and in CI can be difficult in the beginning, too. 

Moreover, spark forces its developer to explicitly think about its distributed computation model to effectively make use of parellism and caching. This is hidden in SQL.

Conclusion
----------

While SQL is the lingua franca for data analysis, it might not be the best choice for everything. SQL is great for dashboarding and BI use cases with simple queries for which the DataFrameAPI is rather over-engineered. However, business critical data pipelines with high complexity and volume are better suited for a DataFrameAPI interface because:

- complexity can be handled with well established concepts from software engineering
- semantic correctness can be guaranteed with dedicated testing frameworks
- developer productivity can be greater due to automation and debugging superiority
- pipeline execution can be better profiled and optimized

Hence, SQL and DataFrameAPI interfaces are complementary with each having its strengths and justification.

Motivation
==========

While pyspark offers great flexibility
