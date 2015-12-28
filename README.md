# InfluxDB.Client

This library makes it easy to be a client for InfluxDB on .NET!

## Usage

The library exposes all operations on InfluxDB and can be used for reading/writing data to/from in two primary ways:
 * Using your own POCO classes.
 * Using dynamic classes.

### Using your own POCO classes.

1. Start by defining a class that represents a row in InfluxDB that you want to store.

```c#
   public class ComputerInfo
   {
      [InfluxTimestamp]
      public DateTime Timestamp { get; set; }

      [InfluxTag( "host" )]
      public string Host { get; set; }

      [InfluxTag( "region" )]
      public string Region { get; set; }

      [InfluxField( "cpu" )]
      public double CPU { get; set; }

      [InfluxField( "ram" )]
      public long RAM { get; set; }
   }
```

On your POCO class you must specify these things:
 * 1 property with the type DateTime as the timestamp used in InfluxDB by adding the [InfluxTimestamp] attribute.
 * 0-* properties with the type string or a user-defined enum with the [InfluxTag] attribute that InfluxDB will use as indexed tags.
 * 0-* properties with the type string, long, double, bool or a user-defined enum with the [InfluxValue] attribute that InfluxDB will use as fields.

Once you've defined your class, you're ready to use the InfluxClient, which is the main entry points to the API:

Here's how to write to the database:

```c#
      private ComputerInfo[] CreateTypedRowsStartingAt( DateTime start, int rows )
      {
         var rng = new Random();
         var regions = new[] { "west-eu", "north-eu", "west-us", "east-us", "asia" };
         var hosts = new[] { "some-host", "some-other-host" };

         var timestamp = start;
         var infos = new ComputerInfo[ rows ];
         for ( int i = 0 ; i < rows ; i++ )
         {
            long ram = rng.Next( int.MaxValue );
            double cpu = rng.NextDouble();
            string region = regions[ rng.Next( regions.Length ) ];
            string host = hosts[ rng.Next( hosts.Length ) ];

            var info = new ComputerInfo { Timestamp = timestamp, CPU = cpu, RAM = ram, Host = host, Region = region };
            infos[ i ] = info;

            timestamp = timestamp.AddSeconds( 1 );
         }

         return infos;
      }
      
      public async Task Should_Write_Typed_Rows_To_Database()
      {
         var client = new InfluxClient( new Uri( "http://localhost:8086" ) );
         var infos = CreateTypedRowsStartingAt( new DateTime( 2010, 1, 1, 1, 1, 1, DateTimeKind.Utc ), 500 );
         await _client.WriteAsync( "mydb", "myMeasurementName", infos, TimestampPrecision.Nanosecond, Consistency.One );
      }
```
