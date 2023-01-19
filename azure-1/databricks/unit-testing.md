# DataBricks Unit Testing

## Steps

In order to be able to perform unit tests, the following steps need to be taken:

1. Import unittest2 and logging
2. Import DataBricksTestCase&#x20;
3. Create TestCase classes, including test functions
4. Run unit tests
5. Evaluate output

### Step 1

```python
import unittest2
import logging
```

### Step 2

```python
class DataBricksTestCase(unittest2.TestCase):

    """Basic common test case for Spark. Provides a Spark context as sc.
    For non local mode testing you can either override sparkMaster
    or set the enviroment property SPARK_MASTER for non-local mode testing."""

    @classmethod
    def getMaster(cls):
        return cs.master

    def setUp(self):
        """Setup a basic Spark context for testing"""
        #self.sc = SparkContext(self.getMaster()) 
        self.sc = sc #Added by René Nadoerp
        self.logger = logging.getLogger('py4j')
        self.logger.setLevel(logging.INFO)

    def tearDown(self):
        """
        Tear down the basic panda spark test case. This stops the running
        context and does a hack to prevent Akka rebinding on the same port.
        """
        self.logger.setLevel(logging.ERROR)
        pass
        #self.sc.stop()
        # To avoid Akka rebinding to the same port, since it doesn't unbind
        # immediately on shutdown
        #self.sc._jvm.System.clearProperty("spark.driver.port")

    def assertRDDEquals(self, expected, result):
        return self.compareRDD(expected, result) == []

    def compareRDD(self, expected, result):
        expectedKeyed = expected.map(lambda x: (x, 1))\
                                .reduceByKey(lambda x, y: x + y)
        resultKeyed = result.map(lambda x: (x, 1))\
                            .reduceByKey(lambda x, y: x + y)
        return expectedKeyed.cogroup(resultKeyed)\
                            .map(lambda x: tuple(map(list, x[1])))\
                            .filter(lambda x: x[0] != x[1]).take(1)

    def assertRDDEqualsWithOrder(self, expected, result):
        return self.compareRDDWithOrder(expected, result) == []

    def compareRDDWithOrder(self, expected, result):
        def indexRDD(rdd):
            return rdd.zipWithIndex().map(lambda x: (x[1], x[0]))
        indexExpected = indexRDD(expected)
        indexResult = indexRDD(result)
        return indexExpected.cogroup(indexResult)\
                            .map(lambda x: tuple(map(list, x[1])))\
                            .filter(lambda x: x[0] != x[1]).take(1)

    def assertDataFrameEqual(self, expected, result, tol=0):
      """Assert that two DataFrames contain the same data.
      When comparing inexact fields uses tol.
      """

      self.assertEqual(expected.schema, result.schema)
      try:
          expectedRDD = expected.rdd.cache()
          resultRDD = result.rdd.cache()
          self.assertEqual(expectedRDD.count(), resultRDD.count())

          def zipWithIndex(rdd):
              """Zip with index (idx, data)"""
              return rdd.zipWithIndex().map(lambda x: (x[1], x[0]))

          def equal(x, y):
              if (len(x) != len(y)):
                  return False
              elif (x == y):
                  return True
              else:
                  for idx in range(len(x)):
                      a = x[idx]
                      b = y[idx]
                      if isinstance(a, float):
                          if (abs(a - b) > tol):
                              return False
                      else:
                          if a != b:
                              return False
              return True
          expectedIndexed = zipWithIndex(expectedRDD)
          resultIndexed = zipWithIndex(resultRDD)
          joinedRDD = expectedIndexed.join(resultIndexed)
          unequalRDD = joinedRDD.filter(
              lambda x: not equal(x[1][0], x[1][1]))
          differentRows = unequalRDD.take(10)
          self.assertEqual([], differentRows)
      finally:
          expectedRDD.unpersist()
          resultRDD.unpersist()
```

### Step 3

```python
from datetime import datetime
from pyspark.sql import Row
from pyspark.sql.types import *

class SimpleSQLTest(DataBricksTestCase):
    """A simple test."""

    def test_empty_expected_equal(self):
        
       
        allTypes = self.sc.parallelize([])
        df = spark.createDataFrame(allTypes, StructType([]))
        
        self.assertDataFrameEqual(df,df)
        
    def test_simple_expected_equal(self):
        allTypes = self.sc.parallelize([Row(
            i=1, s="string", d=1.0, lng=1,
            b=True, list=[1, 2, 3], dict={"s": 0}, row=Row(a=1),
            time=datetime(2014, 8, 1, 14, 1, 5))])
        
        df = allTypes.toDF()
     
         
        self.assertDataFrameEqual(df, df)
        
    def test_simple_expected_equal_dept_emp(self):
    
        # Create the Departments
        department1 = Row(id='123456', name='Computer Science')
        department2 = Row(id='789012', name='Mechanical Engineering')
        department3 = Row(id='345678', name='Theater and Drama')
        department4 = Row(id='901234', name='Indoor Recreation')

        # Create the Employees
        Employee = Row("firstName", "lastName", "email", "salary")
        employee1 = Employee('michael', 'armbrust', 'no-reply@berkeley.edu', 100000)
        employee2 = Employee('xiangrui', 'meng', 'no-reply@stanford.edu', 120000)
        employee3 = Employee('matei', None, 'no-reply@waterloo.edu', 140000)
        employee4 = Employee(None, 'wendell', 'no-reply@berkeley.edu', 160000)

        # Create Department rows
        departments = [department1, department2]

        # Create the Department schema
        fields = [StructField('id', StringType(), nullable = True), StructField('name', StringType(), nullable=True)]
        schema = StructType(fields)

        df_result = spark.createDataFrame(departments, schema)
        df_expected = df_result

        self.assertDataFrameEqual(df_result, df_expected)
     
    def test_simple_expected_unequal_dept_emp(self):
    
        # Create the Departments
        department1 = Row(id='123456', name='Computer Science')
        department2 = Row(id='789012', name='Mechanical Engineering')
        department3 = Row(id='345678', name='Theater and Drama')
        department4 = Row(id='901234', name='Indoor Recreation')

        # Create the Employees
        Employee = Row("firstName", "lastName", "email", "salary")
        employee1 = Employee('michael', 'armbrust', 'no-reply@berkeley.edu', 100000)
        employee2 = Employee('xiangrui', 'meng', 'no-reply@stanford.edu', 120000)
        employee3 = Employee('matei', None, 'no-reply@waterloo.edu', 140000)
        employee4 = Employee(None, 'wendell', 'no-reply@berkeley.edu', 160000)

        # Create Department rows
        departmentsA = [department1, department2, department3]
        departmentsB = [department1, department3, department4]

        # Create the Department schema
        fields = [StructField('id', StringType(), nullable = True), StructField('name', StringType(), nullable=True)]
        schema = StructType(fields)
        
  
        df_result   = spark.createDataFrame(departmentsA, schema)
        df_expected = spark.createDataFrame(departmentsB, schema)

        self.assertDataFrameEqual(df_result, df_expected)

    def test_simple_close_equal(self):
        allTypes1 = self.sc.parallelize([Row(
            i=1, s="string", d=1.0, lng=1,
            b=True, list=[1, 2, 3], dict={"s": 0}, row=Row(a=1),
            time=datetime(2014, 8, 1, 14, 1, 5))])
        allTypes2 = self.sc.parallelize([Row(
            i=1, s="string", d=1.001, lng=1,
            b=True, list=[1, 2, 3], dict={"s": 0}, row=Row(a=1),
            time=datetime(2014, 8, 1, 14, 1, 5))])
        self.assertDataFrameEqual(allTypes1.toDF(), allTypes2.toDF(), 0.1)

    @unittest2.expectedFailure
    def test_simple_close_unequal(self):
        allTypes1 = self.sc.parallelize([Row(
            i=1, s="string", d=1.0, lng=1,
            b=True, list=[1, 2, 3], dict={"s": 0}, row=Row(a=1),
            time=datetime(2014, 8, 1, 14, 1, 5))])
        allTypes2 = self.sc.parallelize([Row(
            i=1, s="string", d=1.001, lng=1,
            b=True, list=[1, 2, 3], dict={"s": 0}, row=Row(a=1),
            time=datetime(2014, 8, 1, 14, 1, 5))])
        self.assertDataFrameEqual(allTypes1.toDF(), allTypes2.toDF(), 0.0001)

    @unittest2.expectedFailure
    def test_very_simple_close_unequal(self):
        allTypes1 = self.sc.parallelize([Row(d=1.0)])
        allTypes2 = self.sc.parallelize([Row(d=1.001)])
        self.assertDataFrameEqual(allTypes1.toDF(), allTypes2.toDF(), 0.0001)

    @unittest2.expectedFailure
    def test_dif_schemas_unequal(self):
        allTypes1 = self.sc.parallelize([Row(d=1.0)])
        allTypes2 = self.sc.parallelize([Row(d="1.0")])
        self.assertDataFrameEqual(allTypes1.toDF(), allTypes2.toDF(), 0.0001)

    @unittest2.expectedFailure
    def test_empty_dataframe_unequal(self):
        allTypes = self.sc.parallelize([Row(
            i=1, s="string", d=1.001, lng=1,
            b=True, list=[1, 2, 3], dict={"s": 0}, row=Row(a=1),
            time=datetime(2014, 8, 1, 14, 1, 5))])
        empty = self.sc.parallelize([])
        self.assertDataFrameEqual(
            allTypes.toDF(),
            self.sqlCtx.createDataFrame(empty, allTypes.toDF().schema), 0.1)
```

### Step 4

```python
import sys
runner = unittest2.TextTestRunner(sys.stdout,verbosity=2)
result = runner.run(unittest2.makeSuite(SimpleSQLTest))
```

Note that the code above is different than usual (python -m unittest test.py). The reason for this is that in DataBricks the unit test is not triggered from the command line, but from within a notebook.

The unit test run will generate output similar to below:

```
test_dif_schemas_unequal (__main__.SimpleSQLTest) ... expected failure
test_empty_dataframe_unequal (__main__.SimpleSQLTest) ... expected failure
test_empty_expected_equal (__main__.SimpleSQLTest) ... ok
test_simple_close_equal (__main__.SimpleSQLTest) ... ok
test_simple_close_unequal (__main__.SimpleSQLTest) ... expected failure
test_simple_expected_equal (__main__.SimpleSQLTest) ... ok
test_simple_expected_equal_dept_emp (__main__.SimpleSQLTest) ... ok
test_simple_expected_unequal_dept_emp (__main__.SimpleSQLTest) ... FAIL
test_very_simple_close_unequal (__main__.SimpleSQLTest) ... expected failure

======================================================================
FAIL: test_simple_expected_unequal_dept_emp (__main__.SimpleSQLTest)
----------------------------------------------------------------------
Traceback (most recent call last):
  File "<command-733711467225637>", line 81, in test_simple_expected_unequal_dept_emp
  File "<command-733711467225632>", line 87, in assertDataFrameEqual
AssertionError: Lists differ: [] != [(1, (Row(id=u'789012', name=u'Mechanical [156 chars]')))]

Second list contains 2 additional elements.
First extra element 0:
(1, (Row(id=u'789012', name=u'Mechanical Engineering'), Row(id=u'345678', name=u'Theater and Drama')))

- []
+ [(1,
+   (Row(id=u'789012', name=u'Mechanical Engineering'),
+    Row(id=u'345678', name=u'Theater and Drama'))),
+  (2,
+   (Row(id=u'345678', name=u'Theater and Drama'),
+    Row(id=u'901234', name=u'Indoor Recreation')))]

----------------------------------------------------------------------
Ran 9 tests in 8.542s

FAILED (failures=1, expected failures=4)
```

### Step 5

```python
if not result.wasSuccessful():
  if len(result.errors) == 1 and not result.failures:
      err = result.errors[0][1]
  elif len(result.failures) == 1 and not result.errors:
      err = result.failures[0][1]
  else:
      err = "multiple errors occurred"
      if not verbose: err += "; run in verbose mode for details"
  raise Exception(err) 
```

## Example Notebook

In the code blocks below and the example notebook at the bottom, unit testing is done using the python unittest2 package. The basis for the code has been the Spark Testing Base package ([https://github.com/holdenk/spark-testing-base](https://github.com/holdenk/spark-testing-base)) developed by Holden Karau.

The code from the spark-testing-base package needed to be altered, because within DataBricks a Spark Context object and a SQL Session object are already available and do not need te be created. In addition, for convenience, the DataBricksTestCase class is defined as a direct subclass of the unittest2.TestCase. It contains assertion-functions that can be used in the testcases.

{% file src="../../.gitbook/assets/DataBricksUnitTest.py" %}
DataBricks Unit Test
{% endfile %}

## Redirect stderr / stdout to file in unittest

A redirect of stdout / stderr requires a file-like object. In order to redirect to a file, open a file (which returns a file like object), and pass this object as a parameter in the TextTestRunner class

```python
#Code requires python 3
from contextlib import redirect_stdout
with open('/dbfs/tmp/unittest.log', 'w') as stdout:
  with redirect_stdout(stdout):
    runner = unittest.TextTestRunner(stdout,verbosity=2)
    result = runner.run(unittest.makeSuite(SimpleSQLTest))
```
