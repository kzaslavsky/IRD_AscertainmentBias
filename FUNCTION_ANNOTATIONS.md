Function Annotations for This Folder

Environment
- BigQuery dataset alias: `CDR`
- GCS bucket: `WORKSPACE_BUCKET`
- I/O helpers use `gsutil` or `google.cloud.storage`

Step2-getPersonIDsAnnotatedWithDisease.ipynb
- getICDcodes(cidlist): Map SNOMED descendants to ICD9/10 via `{CDR}.concept_relationship`; returns mapping DataFrame.
- reconstructICDcodes(cidlist): Two-step SNOMED→SNOMED (Maps to) then SNOMED→ICD mapping; returns DataFrame.
- getICD_personid(ICDcodes_list): Persons with those ICD source concepts (WGS only); merges primary consent date; returns annotated DataFrame.
- calcAgeAtVisit(df): Age at condition visit in years; adds `AgeAtVisit`, `AgeAtVisit_Years`.
- calcAgeAtConsent(df): Age at consent in years; adds `AgeAtConsent`, `AgeAtConsent_Years`.
- keepYoungestVisit(df): Keep min `AgeAtVisit` row per `person_id`.
- TqdmFileWrapper: Wrap file reads to update a tqdm progress bar during upload.
- saveToBucket(df, name, folder="personID_concept"): Save TSV and upload to GCS (progress bar).
- getTable(name, folder="personID_concept"): `gsutil cp` then `pd.read_csv` (tab-delimited).
- getFile(name, folder): `gsutil cp` only.
- makeMultiSQLqeury(ids): Join list into comma-separated string for SQL IN.

Step4-SES_demographics_comorbidities.ipynb
- getTable/getFile/saveToBucket: Same as Step2 (uses gsutil+subprocess for upload).
- widen_comorbidity(df): One-hot comorbidity by `person_id` after mapping codes; ensures standard columns; binary flags.
- assign_comorbidity(code): Regex on ICD-like code to buckets (DM, VitA def, Obesity, Rare, HLD, Neuropsych, ENT, Circulatory/HTN/IHD, GI/Skin/MSK/GU, Congenital/Dactyly, External, Hem/Onc).
- pID_mobidity_master(df): Build ICD-only condition table for given people; apply `assign_comorbidity`; return [wide, long].
- assign_LS_smoke/assign_SES_married(str): Map survey answers to numeric/NaN.
- pID_survey_master(df): Pull SES survey answers for people; compute SES_education/income/healthins/homeown/married + LS_smoke; return [wide, long].
- assign_SES_healthins/education/income/homeown(str): Map SES answers to numeric encodings.
- countVisits(matrix, visits): Merge visit counts as `NumberOfVisits`.
- mergeSurveyforAnno(matrix, src): Mark affected=Yes and merge AgeAtVisit, Sex, race.
- mergeSurveyforUNAnno(matrix, src): Mark affected=No and merge AgeAtConsent, Sex, race.
- makeSexNumeric(df): Map Sex to 0/1 (`sex_numeric`) and drop `Sex`.
- dummyRace(df): One-hot encode `race`.
- bin_outcome(df): Map `affected` Yes/No → 1/0 (`outcome_bin`).
- divideNumVisit(df, amt=10): Scale `NumberOfVisits`.
- divideAge(df, amt=1): Scale `Age`.
- _fit_logit(y, X): Add intercept and fit a logit model, guarding against separation errors.
- _tidy_res(res, var_name): Extract OR/CI/p from a `LogitResults` object for one predictor.
- doLogit_both(df, drop_cols=None, visits_divisor=10, age_divisor=1, compute_vif=False): Run multivariate + univariate logistic fits, optionally scale inputs, return models/results plus optional VIF table.
- tidy_logit_result_both(results_dict, html_path=None, float_fmt="{:0.5f}"): Format the combined 9-column logit summary and optionally emit HTML.

Step3-CalculateDAFs_makeFigs.ipynb
- getTable/getFile/saveToBucket: As above.
- describe_tables(**tables): Per-table age summary (unique persons), male/female counts and ratio.
- annoConsentDate(table): Add `primary_consent_date` via observation (“Consent PII”).
- describe_solution_tables(**tables): Deduplicate by `person_id`, summarize AgeAtConsent_Years and sex counts per solution table.
- calcAgeToday(df): Age today in years (tz-naive handling); validates.
- calcAgeAtConsent(df): As in Step2.
- calc_solved_rate(summary_df): Fill solved_rate per solution subset relative to concept counts using globals for denominators.
- calc_solved_rate_within(summary_df): Same as above but keep concept cohort size as denominator for subsets.
- processConsentDate(df): Pipeline to merge consent dates and compute age at consent.
- generate_merges(base, concept_person, solution_dict): Merge solution person-variants with concept table; assign globals; return created names.
- process_concept_and_solution_tables(...)/_within(...): Describe merged tables and compute solved rates (overall/within); concat all.
- process_concept_solution_gene_summaries(...): Per merged table, per gene: count unique variants and persons (non-null condition).
- process_solution_gene_summaries(...): From solution tables: variant/person counts and allele stats per gene.
- calculate_annotation_rates(sol_gene, con_gene): Person/variant annotation rates by merging summaries.
- calculate_all_annotation_rates(sol_gene, con_gene): Rates by parsed category/inheritance.
- process_solution_variant_summaries(...): Per variant in solutions: person counts and allele stats.
- process_concept_solution_variant_summaries(...): Per variant in merged tables: person counts (annotated only) with labels.
- calculate_all_variant_annotation_rates(sol_var, con_var): Person/allele rates per category.
- concatenate_variant_columns(...)/split_variant_column(...): Build and split `Variant` id `CHROM-POS-REF-ALT`.
- annotate_variants(anno_vars, clinvar, VAT, remove_list=None, filter_canonical=True, drop_original_variant_columns=True): Merge ClinVar and VAT annotations; filter canonical or remove consequences; validates columns.
- procRace(df): Collapse non-informative race to Unknown/Other (final version renames `race_source_value` → `race`).
- replaceRace(df): Map detailed race labels → concise labels.
- get_global_var(name): Return globals()[name].
- makecontable(...): Build 2x2 for solution-in-concept annotated vs not; return table and counts.
- loop_through_fisher(anno_rates, alpha=0.05): Iterate concepts; Fisher OR, Bonferroni p, CI, prevalence, DAR.
- makecontable_bygene(gene, anno_rates, concept, pop_size=317964): 2x2 by gene within a concept.
- loop_through_fisher_bygene(anno_rates, concept, alpha=0.05): Iterate genes; Fisher + metrics; Bonferroni by num genes.
- plot_fishertable_with_arrows(...): Forest-style plot with log axis and arrow caps; saves image.
- plot_fishertable(...): Simpler forest plot; saves image.
- plot_stacked_bars(...)/plot_stacked_bars_bygene(...): Horizontal stacked bars of annotation rates per gene across concepts; draw concept prevalence lines; save.
- plot_gene_summary_aggregated(...): Per gene bar chart (persons; annotate; optionally save).

SuppTables - UpsetPlots.ipynb
- Includes the same table/summary helpers as Step3 (e.g., `describe_solution_tables`, `calc_solved_rate`, `calc_solved_rate_within`, annotation rate pipelines) without the plotting utilities.

Notes
- Ensure `CDR` is set and BigQuery auth is configured.
- Ensure `WORKSPACE_BUCKET` is set for file transfers.
- Many functions assume specific columns (`person_id`, `variant_id`, `GeneID_Symbol`, dates). Validate before reuse.
- Some routines assign results into `globals()`; take care when rerunning cells.
