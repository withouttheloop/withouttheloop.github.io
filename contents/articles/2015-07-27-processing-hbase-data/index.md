---
title: Processing HBASE data with Rx
author: Liam McLennan
date: 2015-07-27 20:32
template: article.jade
---

[HBase](http://hbase.apache.org/) is good at many things but adhoc querying is not one of them. You can query by key, or you can scan over a range of keys. Sometimes it is handy to be able to monitor a table and process rows as they are added. This kind of processing over a stream is what [Reactive Extensions (Rx)](http://reactivex.io/) is good at, so why not combine hbase querying with Rx Observables?

I'll be using the Microsoft Hbase client and an Azure HBase cluster. What I want is to convert an Hbase table into an Rx observable, so that I can filter, transform, group and fold. Step 1 is to fetch my observable:

    var hotRows = HBaseToObservable.Build(new Uri("https://redacted.azurehdinsight.net"), "????", "??????",
    "tablename","start_key")
    .Select(r => new {
    key = Encoding.UTF8.GetString(r.key),
    data = r.values.ToDictionary(
      v => Encoding.UTF8.GetString(v.column),
      v=> Encoding.UTF8.GetString(v.data))});

`hotRows` is now an observable reading forward over the hbase table `tablename` from the key `start_key`. As new records are appended to the table they will be pushed through the observable.

Why do we want an observable view of an hbase table?
--------------

This is most useful for a table storing data such as timeseries data, where we want a live, forward-only stream. With an observable we can do things like:

    hotRows
      .Where(r => r.data["some_column"] == "whatever")
      .GroupBy(r => r.data["colour"])
      .Select(r => r.data["radius"] * Math.PI);

How is it implemented?
----------

In short, it creates an observable, and an hbase table scanner, and pushes rows to the observable. Once it reaches the end of the table it polls every three seconds for new rows.

    public static class HBaseToObservable
    {
    	public static IObservable<CellSet.Row> Build(
    		Uri server,
    		string username,
    		string password,
    		string tableName,
    		string startKey
    		)
    	{
    		var creds = new ClusterCredentials(server, username, password);
    		var client = new HBaseClient(creds);
    		var rowsOut = new ReplaySubject<CellSet.Row>();

    		Task.Run(()=>{
    			GetRows(rowsOut,client,tableName,startKey);
    		});
    		return rowsOut;
    	}

    	private static void GetRows(ReplaySubject<CellSet.Row> rowsOut,
    		HBaseClient client,
    		string tableName,
    		string startKey) {
    		CellSet next = null;
    		string lastKey = "";

    		var scannerInfo = client.CreateScanner(tableName, new Scanner()
    		{
    			batch = 100,
    			startRow = Encoding.UTF8.GetBytes(startKey),
    			endRow = BitConverter.GetBytes(int.MaxValue)
    		});
    		while ((next = client.ScannerGetNext(scannerInfo)) != null)
    			{
    				foreach (var row in next.rows) {
    					lastKey = Encoding.UTF8.GetString(row.key);
    					rowsOut.OnNext(row);
    				}
    			}
    		Task.Delay(TimeSpan.FromSeconds(3))
    			.ContinueWith((t)=>GetRows(rowsOut,client,tableName,lastKey));
    	}
    }
