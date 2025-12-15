# CSV-BlackBox-Writter

This example contains two programs:

- CSV_Writter_Periodicaly : Writes data periodically into .csv files for later retreive in case of errors.
  - This is usefull for example if a user does not want to log into the cloud large data or several signals, but it would like to store locally ( in the device) as some kind of log book.
- Snapshot_Recorder: It uses a circular buffer, storing the last values of a signal "fast" in volatile memory and only write it to a file in an event
  - It would be usefull to simulate a snapshot-like recorder, with very fast data beeing saved in case of an accident for example.
