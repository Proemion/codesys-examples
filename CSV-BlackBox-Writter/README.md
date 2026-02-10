# CSV-BlackBox-Writer

This example contains two programs:

- CSV_Writer_Periodically : Writes data periodically into .csv files for later retrieve in case of errors.
  - This is useful for example if a user does not want to log into the cloud large data or several signals, but it would like to store locally ( in the device) as some kind of log book.
  - The .csv files are stored under /var/opt/codesys/PlcLogic/CSV_BALCKBOX
- Snapshot_Recorder: It uses a circular buffer, storing the last values of a signal "fast" in volatile memory and only write it to a file in an event
  - It would be useful to simulate a snapshot-like recorder, with very fast data being saved in case of an accident for example.
  - The .csv files are stored under /var/opt/codesys/PlcLogic/CSV_Snapshot

Since the PlcLogic folder can be accessed from CODESYS IDE, the easier way to retrieve the files is to connect ( over remote machine tunnel) using the CODESYS gateway and IDE. Then navigate to Device --> Files and use the arrows to move files between your PC and the device.
