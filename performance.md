# Performance

## Transaction Bundle Upload

### Test Data

Generated 10,000 patients with Synthea master branch with Git SHA `4fed9eaf` and standard configuration using `./run_synthea -p 10000` resulting in 9.9 GiB of JSON files.

### Test System

MacBook Pro \(Retina, 15-inch, Mid 2015\) 2,5 GHz Intel Core i7, 16 GB RAM. Blaze version 0.6-alpha48.

### Upload Method

Command line tool `blazectl` on a  with concurrency of 8.

```text
Uploads       [total, concurrency]     11647, 8
Success       [ratio]                  100 %
Duration      [total]                  2h42m43s
Latencies     [mean, 50, 95, 99, max]  6.695s, 3.502s, 28.023s, 36.342s 2m45.764s
Bytes In      [total, mean]            828.40 MiB, 72.83 KiB
Bytes Out     [total, mean]            9.88 GiB, 889.33 KiB
Status Codes  [code:count]             200:11647
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

The size of the Datomic database \(free version\) on disk after the import was 7.8 GiB.



