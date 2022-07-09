=========
Rationale
=========

Spark is a state-of-the-art distributed computation engine. It scales
from single local machine to clusters of thousand nodes. It offers both a
SQL and DataFrameAPI interface to translate business logic into a logical
representation. It has bindings for Python, Java and R.

Why Spark?
==========

Spark stands out in contrast to classical SQL-only engines in regard to the
following points:

1. **Managing complexity**: Data pipelines often need to represent highly complex
business demands such as multiple joins, aggregates, time series analysis and
more. Handling such complexity requires a toolset that provides proper
abstractions in order to simplify and modularize responsibilities.

Using a general purpose language (e.g. python) in conjunction with spark's
dataframe API to formulate business demands, opens the door to leverage well
known software development concepts such as inheritance, composition,
introspection and higher order functions. These techniques make it easy to
structure, reuse and dynamically generate code with great benefits for
development and maintenance of data pipelines.

In contrast, an SQL-only interface is solely string based and lacks the before
mentioned functionality. Hence, SQL statements often become very complex,
cluttered and hard to reason about.

Frameworks like dbt do a great job in
trying to tackle these issues with macros for example. However, they do not
solve the core problem of not having a proper object representation of entities
such as dataframes and their available methods. Dbt macros still generate a
string and do not have an inherent understanding of what objects they deal
with.

2. **Testing**: Spark can be run locally in a standalone mode on a
developer's machine or in a continuous integration (CI) environment. Using pyspark
as a binding for spark allows to employ the rich testing ecosystem of python
(e.g. pytest).

Hence, unit tests on separate logical transformations or even end-to-end tests
in scope of the date pipeline are easily possible to ensure semantic correctness.
Additionally, using mocked test data or property-based testing help to cover
edge cases of unexpected input values. Moreover, once the test suite is set up,
refactorings can be safely done.

Altogether, this allows software engineering best practices to be applied to
the development of data pipelines.

In contrast, there does not seem to be a competitive equivalent in the
SQL-only space. Many SQL engines cannot be run locally or in a CI
environment without cloud access (e.g. Snowflake). Each time
you test, you need to spin up cloud compute. Each time you test, you will
likely need to create lots of temporary input and output tables.

More drastically, there is no dedicated testing framework. With pytest for example,
you can combine and manage your tests within a test suite with the ability to
mark or parametrize tests, collect test results, show detailed differences for
failing tests and more.

Dbt attempts to solve some of the issues while providing tests, too. However,
these tests mainly address shape and integrity (e.g. not null). Even with a
framework like ``great expectations``, you do not test the semantic correctness
but rather focus on data quality measures.

3. **Debugging**: TBD

4. **Declarative and imperative approach**: SQL is declarative by heart. The
user describes *what* result is required. The underlying engine determines
*how* this result needs to computed for best performance.

This is well suited in more than 95% of the cases for two reasons. First, the
typical end user only cares about the result. Second, vendors of SQL computation
engines have decades of experience in optimizing queries und generating
physical execution plans.

However in the remaining 5%, especially with increasing complexity, even well
optimized engines will eventually fail. This is especially true in a
distributed environment where the correct partitioning of the data is key. If
not handled correctly, data skew and inappropriate parallelism might result in
insufficient memory (with disk spills or even out of memory exception) and low
CPU utilization.

Spark's dataframe API also provides the possibility to imperatively adjust the
computation graph. More concretely, within the declarative frame of high level
SQL and dataframe API, it is possible to dictate *how* the computation
plan is generated on a lower level. For example, the partitioning of the data
can be manually enforced via ``repartition``. Moreover, intermediate results
can be explicitly cached to prevent expensive re-computations via ``cache``.
And last but not least, join strategies may directly provided via join hints
(e.g. broadcast joins).



Motivation
==========

While pyspark offers great flexibility