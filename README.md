# PerformanceMarkers.NET

A lightweight, extremely easy-to-use performance reporting library for the .NET Framework. PerformanceMarkers is designed to be integrated into applications whose run times must be measured in *milliseconds*.

PerformanceMarkers.NET supports the **Integrated Performance Metrics Approach,** which consists of the 3 following phases:

* **Exposure.**  This phase determines where any bottlenecks are - and yields a full performance profile of your application.
* **Tuning Verification.**  This is the phase where you make a single, isolated change to an application with the intent to alter its performance properties.  You then prove (or disprove) the performance impact of that change by **marking** the application again and obtaining a performance report.  The process of marking and then reporting is called **profiling**.
* **Convergence.** During this phase, you are methodically increasing your application's performance over time as a fundamental part of your software development process.  This is the most "mature" phase.

How this all works is simple: marker statements are inserted directly into your code - very similar to how you do logging.  In effect, measuring performance becomes a function of the application itself.

We believe that collecting these metrics should be a fundamental part of the software development process due to the enormously positive quality impact it has.

Since the performance metrics collection is integrated into your program - and at some point your program may be deployed to a production environment - we provide ways to disable this library.  This makes it similar to a logging framework where you want it to be less "chatty" in production scenarios.

## Usage

Let's jump right in and show you some code, with some comments to explain what is going on.  We will go into much further detail below.

	//
	// REFERENCE THE PERFORMANCE MARKERS NAMESPACE.
	//
	using PerformanceMarkers;

	//
 	// CREATE A PERFORMANCE MARKER NAMED 'MyProcessName'.
	//
	Marker CreatedMarker = MarkerFactory.StartMarker("ProcessName");

	//
 	// START TIMING AN ACTIVITY NAMED 'Query'.
	//
	CreatedMarker.Start("Query");

	//
	// DO THE ACTIVITY YOU WANT TO TIME.
	//
	Thread.Sleep(577);

	//
	// STOP TIMING THE QUERY ACTIVITY.
	//
	CreatedMarker.End("Query");

	//
	// STOP TIMING THE MARKER ACTIVITY.
	//
	CreatedMarker.End();

	//
	// CREATE A PLAIN-TEXT PERFORMANCE REPORT.
	//
	string PerformanceReport = MarkerReportFactoryProvider.CreateReportFactory().CreateReport(CreatedMarker);

	//
	// VIEW THE REPORT BY LOGGING IT OR DUMPING IT TO THE CONSOLE.
	//
	Console.WriteLine(PerformanceReport);

## Sample Report

Here is a sample report from a database-intensive financial analytics program that summaries about 100,000 activities:

	RanksApp [total: 164,655.4 ms; hidden: 2.0].
		+ MaxRankIdQuery [count: 1; total: 155.0 ms]
		+ AllCompaniesQuery [count: 1; total: 2,058.1 ms]
		+ Transaction [count: 5; total: 162,440.3 ms; avg: 32,488.058; max: 34,607.0; min: 28,140.6]
		Transaction [total: 33,297.9 ms; hidden: 20.0].
			+ BeginTransaction [count: 1; total: 4.0 ms]
			+ Loader.Load [count: 5,000; total: 23,685.3 ms; avg: 4.737; max: 69.0; min: 4.0]
			+ _CompanyCache.GetByTickerOrSEDOL6 [count: 5,000; total: 6.0 ms; avg: 0.001; max: 1.0; min: 0.0]
			+ TargetRankQuery.List [count: 5,000; total: 4,026.2 ms; avg: 0.805; max: 27.0; min: 0.0]
			+ SessionSaveNewRank [count: 5,000; total: 214.0 ms; avg: 0.043; max: 39.0; min: 0.0]
			+ Commit [count: 1; total: 5,342.3 ms]
		(THE REMAINING 4 TRANSACTIONS WERE OMITTED FOR BREVITY.)

This report gives us a summary of how long each query and transaction took to complete.  Let's break this report down more and go through each line:

	RanksApp [total: 164,655.4 ms; hidden: 2.0].

This line tells us that the 'RanksApp' took a total of 164,655.4 ms to run - about 165 seconds, and that there was 2.0 ms of hidden processing time.  **Hidden processing time** is the difference between a parent activity's total time and the sum of its child activity total times.  This is processing time that was not accounted for by your markers - and you generally want this to be a very low, negligible number.

		+ MaxRankIdQuery [count: 1; total: 155.0 ms]
		+ AllCompaniesQuery [count: 1; total: 2,058.1 ms]

These are **child activity summaries**.  Summaries are always prefixed with a '+'.  These summaries tell us that each query was executed once (count: 1) along with the total processing times for each.

		+ Transaction [count: 5; total: 162,440.3 ms; avg: 32,488.058; max: 34,607.0; min: 28,140.6]

This is a summary for a child activity that observed more than once.  In this case, a transaction that ran 5 times (count: 5).  We also include the total, average, maximum, and minimum times for each child activity.

		Transaction [total: 33,297.9 ms; hidden: 20.0].
			+ BeginTransaction [count: 1; total: 4.0 ms]
			+ Loader.Load [count: 5,000; total: 23,685.3 ms; avg: 4.737; max: 69.0; min: 4.0]
			+ _CompanyCache.GetByTickerOrSEDOL6 [count: 5,000; total: 6.0 ms; avg: 0.001; max: 1.0; min: 0.0]
			+ TargetRankQuery.List [count: 5,000; total: 4,026.2 ms; avg: 0.805; max: 27.0; min: 0.0]
			+ SessionSaveNewRank [count: 5,000; total: 214.0 ms; avg: 0.043; max: 39.0; min: 0.0]
			+ Commit [count: 1; total: 5,342.3 ms]

This is one of the 5 transactions that were run.  It has its own set of child activity summaries.  Thus, performance reports are hierarchical.  Notice how it is summarizing several thousand children for us.  In this case, the activity named 'Loader.Load' is consuming most of the processing time (23,685.3 ms out of 33,297.9 ms).  Thus, this real-world example has likely exposed what we need to examine next.

## Performance Reports

Collecting performance points is the easy part. Creating easy-to-read reports with summaries is what PerformanceMarkers does well.


You can export to the following media types:

* **Text** - a human-readable hierarchy of activities with summaries of how long each activity took to complete.  See above for a sample report.
* **XML** - same as the text format, except you can also easily load it using any XML parser.  This is useful when you may need to write a script that aggregates several performance reports.

## Concurrent Performance Tracking

Instances of the Marker class are not thread-safe.  Ensure that exactly 1 thread is accessing a marker at a time.

Instances of the  MarkerFactory class are not thread-safe, either.  However the following statement is thread-safe because it creates an instance of a marker factory and uses it. 

	MarkerReportFactoryProvider.CreateReportFactory(MarkerReportFactoryType.Xml).CreateReport(CreatedMarker);


Now that we know that markers and marker factories are not thread-safe, we can develop a strategy 

## Performance Reports - XML

There are 2 ways to get a performance report in XML format.  You can generate the report directly as a string:

	string PerformanceReport = MarkerReportFactoryProvider.CreateReportFactory(MarkerReportFactoryType.Xml).CreateReport(CreatedMarker);

You can also write to a stream:

	using (FileStream TargetStream = new FileStream("path/to/file.xml", FileMode.Create))
	{
		MarkerReportFactory CreatedFactory = MarkerReportFactoryProvider.CreateReportFactory(MarkerReportFactoryType.Xml)
		CreatedFactory.TargetStream = TargetStream;
		CreatedFactory.Marker = CreatedMarker;
		CreatedFactory.WriteReport();
	}

The output will have a structure like this:

	<Activity Name="RanksApp" Total="48,456.8" Hidden="6.0">
	  <Summaries Count="4">
	    <Summary Name="MaxRankIdQuery" Count="1" Total="156.0" Max="156.0" Avg="156.009" Min="156.0" />
	    <Summary Name="AllCompaniesQuery" Count="1" Total="2,064.1" Max="2,064.1" Avg="2,064.118" Min="2,064.1" />
	    <Summary Name="Loader.Load" Count="5" Total="34.0" Max="34.0" Avg="6.800" Min="0.0" />
	    <Summary Name="Transaction" Count="5" Total="46,196.6" Max="9,990.6" Avg="9,239.328" Min="7,705.4" />
	  </Summaries>
	  <ChildActivities>
	    <Activity Name="Transaction" Total="9,568.5" Hidden="4.0">
	      <Summaries Count="6">
	        <Summary Name="BeginTransaction" Count="1" Total="4.0" Max="4.0" Avg="4.000" Min="4.0" />
	        <Summary Name="_CompanyCache.GetByTickerOrSEDOL6" Count="5,000" Total="3.0" Max="1.0" Avg="0.001" Min="0.0" />
	        <Summary Name="TargetRankQuery.List" Count="5,000" Total="3,832.2" Max="60.0" Avg="0.766" Min="0.0" />
	        <Summary Name="SessionSaveNewRank" Count="5,000" Total="226.0" Max="39.0" Avg="0.045" Min="0.0" />
	        <Summary Name="Loader.Load" Count="4,999" Total="131.0" Max="22.0" Avg="0.026" Min="0.0" />
	        <Summary Name="Commit" Count="1" Total="5,368.3" Max="5,368.3" Avg="5,368.307" Min="5,368.3" />
	      </Summaries>
	      <ChildActivities />
	    </Activity>
	    <Activity Name="Transaction" Total="9,415.5" Hidden="10.0">
	      <Summaries Count="6">
	        <Summary Name="BeginTransaction" Count="1" Total="1.0" Max="1.0" Avg="1.000" Min="1.0" />
	        <Summary Name="_CompanyCache.GetByTickerOrSEDOL6" Count="5,000" Total="10.0" Max="1.0" Avg="0.002" Min="0.0" />
	        <Summary Name="TargetRankQuery.List" Count="5,000" Total="3,749.2" Max="3.0" Avg="0.750" Min="0.0" />
	        <Summary Name="SessionSaveNewRank" Count="5,000" Total="186.0" Max="1.0" Avg="0.037" Min="0.0" />
	        <Summary Name="Loader.Load" Count="4,999" Total="142.0" Max="1.0" Avg="0.028" Min="0.0" />
	        <Summary Name="Commit" Count="1" Total="5,317.3" Max="5,317.3" Avg="5,317.304" Min="5,317.3" />
	      </Summaries>
	      <ChildActivities />
	    </Activity>
	    <Activity Name="Transaction" Total="9,516.5" Hidden="9.0">
	      <Summaries Count="6">
	        <Summary Name="BeginTransaction" Count="1" Total="0.0" Max="0.0" Avg="0.000" Min="0.0" />
	        <Summary Name="_CompanyCache.GetByTickerOrSEDOL6" Count="5,000" Total="0.0" Max="0.0" Avg="0.000" Min="0.0" />
	        <Summary Name="TargetRankQuery.List" Count="5,000" Total="3,843.2" Max="85.0" Avg="0.769" Min="0.0" />
	        <Summary Name="SessionSaveNewRank" Count="5,000" Total="161.0" Max="1.0" Avg="0.032" Min="0.0" />
	        <Summary Name="Loader.Load" Count="4,999" Total="152.0" Max="1.0" Avg="0.030" Min="0.0" />
	        <Summary Name="Commit" Count="1" Total="5,351.3" Max="5,351.3" Avg="5,351.306" Min="5,351.3" />
	      </Summaries>
	      <ChildActivities />
	    </Activity>
	    <Activity Name="Transaction" Total="9,990.6" Hidden="3.0">
	      <Summaries Count="6">
	        <Summary Name="BeginTransaction" Count="1" Total="1.0" Max="1.0" Avg="1.000" Min="1.0" />
	        <Summary Name="_CompanyCache.GetByTickerOrSEDOL6" Count="5,000" Total="8.0" Max="1.0" Avg="0.002" Min="0.0" />
	        <Summary Name="TargetRankQuery.List" Count="5,000" Total="3,737.2" Max="3.0" Avg="0.747" Min="0.0" />
	        <Summary Name="SessionSaveNewRank" Count="5,000" Total="185.0" Max="3.0" Avg="0.037" Min="0.0" />
	        <Summary Name="Loader.Load" Count="4,999" Total="124.0" Max="1.0" Avg="0.025" Min="0.0" />
	        <Summary Name="Commit" Count="1" Total="5,932.3" Max="5,932.3" Avg="5,932.339" Min="5,932.3" />
	      </Summaries>
	      <ChildActivities />
	    </Activity>
	    <Activity Name="Transaction" Total="7,705.4" Hidden="3.0">
	      <Summaries Count="6">
	        <Summary Name="BeginTransaction" Count="1" Total="1.0" Max="1.0" Avg="1.000" Min="1.0" />
	        <Summary Name="_CompanyCache.GetByTickerOrSEDOL6" Count="4,280" Total="2.0" Max="1.0" Avg="0.000" Min="0.0" />
	        <Summary Name="TargetRankQuery.List" Count="4,280" Total="3,217.2" Max="2.0" Avg="0.752" Min="0.0" />
	        <Summary Name="SessionSaveNewRank" Count="4,280" Total="149.0" Max="1.0" Avg="0.035" Min="0.0" />
	        <Summary Name="Loader.Load" Count="4,280" Total="102.0" Max="2.0" Avg="0.024" Min="0.0" />
	        <Summary Name="LastCommit" Count="1" Total="4,231.2" Max="4,231.2" Avg="4,231.242" Min="4,231.2" />
	      </Summaries>
	      <ChildActivities />
	    </Activity>
	  </ChildActivities>
	</Activity>

## Configuration

The initial system configuration is targeted for development environments and is defined in MarkerConfigReference.cs:

	MarkerConfig.Type = MarkerType.Enabled;
	MarkerConfig.FailureMode = MarkerFailureMode.HighlyVisible;
	MarkerConfig.ReportFactoryType = MarkerReportFactoryType.PlainText;

By default, markers are enabled - which means they collect start and end times for activities.  A disabled marker does not save this data.

The failure mode is set to be "highly visible" by default - which means the framework will throw exceptions and possibly disrupt the application.  This helps discover and fix any performance marker issues before your application gets to production.

In addition, the framework will produce plain-text reports unless configured otherwise.  This makes it very convenient to output reports directly to the log so it can be reviewed.

PerformanceMarkers.NET has a "configurator" object that configures the marker framework and automatically propagates an updated configuration.  This lets the administrator change the configuration without restarting the application.

The configuration is an XML file that looks like this:

	<MarkerConfig>
		<MarkerType>Disabled</MarkerType>
		<MarkerFailureMode>CompletelyHidden</MarkerFailureMode>
		<MarkerReportFactoryType>PlainText</MarkerReportFactoryType>
	</MarkerConfig>

You then use the XmlConfigurator class to load the configuration.

	//
	// CONFIGURES THE PERFORMANCE MARKERS FRAMEWORK AND WATCHES FOR CHANGES.
	//
	XmlConfigurator.ConfigureAndWatch("path/to/markers-config.xml");

When you use the MarkerFactory to create a marker - the factory pays close attention to the current configuration. 

## Development & Testing Configuration

The recommended configuration for development and testing environments is this:

	<MarkerConfig>
		<MarkerType>Enabled</MarkerType>
		<MarkerFailureMode>HighlyVisible</MarkerFailureMode>
		<MarkerReportFactoryType>PlainText</MarkerReportFactoryType>
	</MarkerConfig>

A development and testing environment is an environment where performance reporting is accepted since the purpose of these environment is to measure application quality.

## Production Configuration

In production-like environments, performance and stability are favored over performance reporting.  Therefore, we recommended disabling the marker framework and suppressing any exceptions or other behavior that would be disruptive to the application.

The following configuration file is suitable for production-like environments:

	<MarkerConfig>
		<MarkerType>Disabled</MarkerType>
		<MarkerFailureMode>CompletelyHidden</MarkerFailureMode>
		<MarkerReportFactoryType>PlainText</MarkerReportFactoryType>
	</MarkerConfig>
