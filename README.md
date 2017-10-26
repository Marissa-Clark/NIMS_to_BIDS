# NIMS_to_BIDS Conversion

## Usage

```python NIMS_to_BIDS.py projectName ```
(requires a completed BIDS_info.xlsx file)

## Summary

Takes neuroimaging downloaded from the Neurobiological Image Managament System at Stanford University (https://cni.stanford.edu/nims/) and converts it to BIDS (http://bids.neuroimaging.io/)

This version of NIMS-to-BIDS is intended to work through the command line on Sherlock. The script:
* takes a single argument---the project name---and searches for scans in $PI_SCRATCH/[projectName]/NIMS_data
* checks that each scan sessions matches the protocol listed in the BIDS_info.xlsx file
* places the BIDS-formatted data in $PI_HOME/[projectName]/BIDS_data.


```
NIMS Data Format (e.g)

|-- ProjectFilename
    |-- BIDS_info.xlsx
    |-- NIMS_data
        |-- 20170101_14000
            |-- 1_1_3Plane_Loc_fgre
                |-- 15238_1_1.nii.gz
            |-- 2_1_ASSET_calibration
                |-- 15238_2_1.nii.gz
            |-- 4_1_BOLD_EPI_29mm_2sec
                |-- 15238_4_1.nii.gz
            |-- 5_1_BOLD_EPI_29mm_2sec
                |-- 15238_5_1.nii.gz
            |-- 6_1_BOLD_EPI_29mm_2sec
                |-- 15238_6_1.nii.gz
            |-- 7_1_BOLD_EPI_29mm_2sec
                |-- 15238_7_1.nii.gz
            |-- 8_1_T1w_9mm_BRAVO
                |-- 15238_8_1.nii.gz

BIDS Data Format (e.g) http://bids.neuroimaging.io/

|-- ProjectFilename
    |-- BIDS_data
        |-- participants.tsv
        |-- task-empathy_bold.json
        |-- dataset_description.json
        |-- sub-101
            |-- anat
                |-- sub-101_T1w.nii.gz
            |-- func
                |-- sub-101_task-empathy_run-01_bold.nii.gz
                |-- sub-101_task-empathy_run-02_bold.nii.gz
                |-- sub-101_task-empathy_run-03_bold.nii.gz
                |-- sub-101_task-empathy_run-04_bold.nii.gz
```

## BIDS_info.xlsx

NIMS_to_BIDS uses a BIDS_info.xlsx file as a reference between NIMS and BIDS formatting. 

**dataset**: Dataset information (following BIDS convention for `dataset_information.json`)

 * Name (database name)
 * (Optional) License, authors, Acknowledgments, HowToAcknowledge, Funding, ReferencesAndLinks, DatasetDOI

**tasks**: Task information

* BIDS_scan_title
  * If you would not like a scan to be included in your BIDS folder, leave blank
  * For task-related fMRI, start the scan name with "task-" (e.g., task-tomloc)
* TaskName
    * e.g., "Theory of Mind functional localizer" for "tomloc"
* RepetitionTime
* EchoTime
  * (Optional) Used for fieldmap correction. If not applicable, leave blank

**participants**: Participant information

  * nims_title (the data and a 5 digit id number)
  * participant_id
  * (Optional) age, gender

**protocol**: Protocol information

* participant_id
  * The session title. "default" for default protocol (see notes below), specific session IDs for custom protocols.
* sequence_no
  * The number of the sequence, according to the BIDS protocol. Must correspond to folder number in NIMS_data.
  * TODO: In this current version, you must specify sequence numbers even for the default protocol. That means that you must specify custom protocols for participants if their sequence numbers are off by a constant shift, even if the sequences are in the same order as the default protocol. (For example, if you run two shim sequences for one subject, rather than one, then the part of the protocol we're copying to BIDS will start at sequence #5, not sequence #4. In the current version, you'd need to specify a custom protocol for this subject. In future versions, this may not be necessary.)
* NIMS_scan_title
  * e.g. 3Plane_Loc_fgre
* BIDS_scan_title 
  * Following the bids naming convention. If you would not like this to go to your bids folder, leave blank
  * By convention: use "fieldmap" for fieldmaps, "task-[TASKNAME]" for functional tasks (even resting state fMRI, in which case you would use "task-rest"), and modality (e.g., "T1w") for structural images.
  * Make sure that task-based fMRI names match BIDS_scan_title in "tasks"
* run_number
  * (optional) if applicable, if not, leave blank
* sequence type
  * Data type. Use BIDS convention (e.g., "anat", "func", "fmap")

**Custom Protocols**

If a specifc participant recieved a different protocol than the 'default' protocol, they can be added under participant_id. For an example, view the BIDS_info.xlsx example file. 



**A note on sequence-specific fields:**
The protocol sheet is totally customizable. If you add new columns to the protocol sheet, the script will use it to add custom meta-data to sequences that have a non-blank value. In the example BIDS_info sheet, there are two such custom fields, "Units" and "IntendedFor", which are used specifically for fieldmap sequences. If no changes are made to the script, these will be saved "as-is" as meta-data. For example, it will create a JSON file specifically for the sequence in line 6 that contains the following information:

```
{"Units": "Hz",
 "IntendedFor": "5 6"}
```

In principle, you can add *any* new meta-data to the protocols, as long as: (1) the column name matches a field in the BIDS specification, and (2) the content of the cell can be saved to a JSON file as is. Some additional changes will need to be made if you need to transform the input (e.g., the sequence numbers in "IntendedFor") to conform to the BIDS specification.

