# Performance

## Transaction Bundle Upload

### Test Data

Generated 10,000 patients with Synthea master branch with Git SHA `4fed9eaf` and standard configuration using `./run_synthea -p 10000` resulting in 9.9 GiB of JSON files.

### Test System

MacBook Pro \(Retina, 15-inch, Mid 2015\) 2,5 GHz Intel Core i7, 16 GB RAM. Blaze version 0.8.0-beta.3.

### Start Script

```bash
DB_DIR=~/blaze-data/db java -jar blaze-0.8.0-beta.3-standalone.jar -m blaze.core
```

### Relevant Startup Log Output

```text
Init resource indexer executor with 4 threads
Init RocksDB block cache of 128 MB
Init RocksDB statistics
Open RocksDB key-value store in directory `~/blaze-data/db` with options: {:max-background-jobs 4, :compaction-readahead-size 0}
Create resource cache with a size of 10000 resources
Open local transaction log with a resource indexer batch size of 1.
Init FHIR transaction interaction executor with 8 threads
Init JSON parse executor with 8 threads
Init server executor with 8 threads
JVM version: 11.0.7
Maximum available memory: 4096 MiB
Number of available processors: 8
Successfully started Blaze version 0.8.0-beta.3 in 14.2 seconds
```

### Upload Method

Command line tool `blazectl` on a with concurrency of 8.

```text

```

The upload resulted in the following resource counts:

| Metric | Count |
| :--- | :--- |
| AllergyIntolerance | 5,624 |
| CarePlan | 35,642 |
| Claim | 508,655 |
| Condition | 84,133 |
| DiagnosticReport | 142,743 |
| Encounter | 402,061 |
| ExplanationOfBenefit | 402,061 |
| Goal | 26,496 |
| ImagingStudy | 7,419 |
| Immunization | 144,534 |
| MedicationAdministration | 4,688 |
| MedicationRequest | 106,594 |
| Observation | 2,010,002 |
| Organization | 33,897 |
| Patient | 11,645 |
| Practitioner | 33,896 |
| Procedure | 322,732 |

The size of the database directory after the import was ?? GiB.

