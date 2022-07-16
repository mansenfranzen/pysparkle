=========
Rationale
=========

Spark is a state-of-the-art distributed computation engine. It scales
from single local machine to clusters of thousand nodes. It offers both a
SQL and DataFrameAPI interface to translate business requirements into a logical
representation. It has bindings for Python, Java and R.

Spark's DataFrameAPI vs. Cloud native SQL engines
=================================================

This article is an opiniated comparison between Spark's DataFrameAPI and cloud native SQL engines (e.g. Snowflake, Redshift, Athena and alike). It aims to outline some of the benefits of using Spark and its DataFrameAPI in contrast to seemingly more easier to use cloud native SQL engines.

Managing complexity
-------------------

Data pipelines often need to represent highly complex transformations such as time series analysis. This typically inludes multiple joins, groupby-aggregates, sophisticated window functions and more. Handling such complexity requires simplification via appropriate abstractions.

Spark's DataFrameAPI is embedded in a general purpose language like python. Entities 
such as dataframes or transformations like a join gain an object representation which enables direct interaction with them. Hence, it is called DataFrame *Application Programming Interface* (API). 

Importantly, this allows to leverage programming concepts such as inheritance, composition, introspection and higher order functions to build up data pipelines. These techniques are a toolbox to create abstractions to handle complexity. More concretely, they enable data engineers to structure, reuse and dynamically generate transformation logic.

In contrast, an SQL-only interface is not embedded in a programming language and remains string based. Therefore, the before mentioned programming techniques can't be applied. Due to the lack of abstraction, SQL statements often tend to result in very complex, cluttered and hard to reason strings. 

Frameworks like dbt address this shortcomings by introducing so called macros. They allow to delegate SQL string generation to arbitrary functions. However, they do not solve the core problem of not having a proper object representation of entities such as dataframes and their corresponding methods. Dbt macros still generate a string and do not have an inherent understanding of what objects they deal with. 

Testing
-------

Spark can be run locally in a standalone mode on a developer's machine or in a continuous integration (CI) environment. Moreover, since the DataFrameAPI is embedded in a programming language, it allows to employ rich testing ecosystems such as pytest or junit.

Accordingly, data engineers can easily develop unit tests on individual transformations or integration tests to ensure semantic correctness of the data pipeline. Moreover, using more sophisticated techniques like property-based testing or mutation tests help to revail edge cases. Once the test suite is set up, refactorings can be safely done. 
Taken together, this allows software engineering best practices to be applied to the development of data pipelines.

In contrast, there does not seem to be a competitive equivalent in the cloud native sql space. Many SQL engines cannot be run locally or in a CI environment without cloud access. Each time you test, you need to spin up cloud compute. Each time you test, you will likely need to create lots of temporary input and output tables. Cloud SQL testing is both slower and likely more expensive than local or CI based testing.

More drastically, there is no dedicated testing framework in the sql space (this is not specific to cloud based sql engines but is valid for sql engines in general). For example, pytest adds the following capabilities for testing your spark data pipelines:

- dynamically generate tests via parametrization
- combine and manage tests into categories via marks or test suits
- run only relevant tests via flexible filters
- provide meaningful differences in failed tests
- centrally collect test results and rerun failed tests

Dbt attempts to solve some of these issues while providing a minimal testing framework. However,
this is in no sense comparable with a dedicated testing framework like pytest. Moreover, dbt tests mainly focus on shape, integrity and related data quality measures (e.g. not null). It can be further extented with ``great expectations`` which offers a huge suite of additional tests. However, it still fails to test the semantic correctness of a data pipeline.

Debugging
---------

Finding and solving bugs in a spark data pipeline follows the same principle as in traditional software development. Using a debugger of your favourite IDE allows you to step through every line of your code while being able to interactively follow and modify the state of the data pipeline for full observability. Even though Spark's execution model is lazy, you can always use a ``show`` or ``toPandas`` on intermediate dataframe representations to inspect results.

Interactive debugging like stopping execution, inspecting and interacting with the state, and resuming execution, is not possible with SQL. For Spark's DataFrameAPI, the smallest observable unit is a line of code. For SQL, it is a view or table. Accordingly, to properly debug in SQL, one needs to create lots of intermediate views or tables. 


Declarative and imperative paradigm
-----------------------------------

SQL is declarative by heart. The user describes *what* result is required. The underlying engine determines *how* this result needs to computed for best performance.

This is well suited in more than 95% of the cases for two reasons. First, the
typical end user only cares about the result. Second, vendors of SQL computation
engines have decades of experience in optimizing queries und generating the best
physical execution plans.

However in the remaining 5%, especially with increasing complexity, even well
optimized engines will eventually fail. This is especially true in a
distributed environment where the correct partitioning of the data is key. If
not handled correctly, data skew and inappropriate parallelism might result in
insufficient memory (with disk spills or even out of memory exception) and low
CPU utilization.

Spark's DataFrameAPI also provides the possibility to imperatively adjust the
computation graph. More concretely, DataFrameAPI allows to dictate *how* the computation
plan is generated on a lower level. For example, the partitioning of the data
can be manually enforced via ``repartition``. Moreover, intermediate results
can be explicitly stored via ``cache`` to prevent expensive re-computations.
And last but not least, join strategies may directly provided via join hints
(e.g. broadcast joins).


Conclusion
----------

While SQL is the lingua france for data analysis, it might not be the best choice for complex data pipelines. SQL is great for dashboarding and BI use cases with simple queries based on already aggregated data. However, business critical data pipelines with large amounts of data and high complexity might be better handled with a DataFrameAPI engine because:

- complexity can be handled with known concepts from software engineering
- semantic correctness can be guaranteed with dedicated testing frameworks
- developer productivity is greater due to automation and better debugging support
- pipeline execution can be better profiled and optimized


Motivation
==========

While pyspark offers great flexibility
