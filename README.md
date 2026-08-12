# Palace Quicksight Resources


This repository contains exported AWS Quicksight assets defining templates and datasets that were exported  with the [palace-quicksight tool](https://github.com/ThePalaceProject/palace-quicksight).  
Please see the documentation there for learning how export, version, and import Palace Quicksight dashboards.

## Dataset refresh configuration

The `assets/data-sets/*-refresh-schedules.json` and `*-refresh-props.json` files are the source of
truth for the SPICE refresh schedules and incremental-refresh look-back windows. They are applied by
the palace-quicksight tool on every import, replacing whatever is configured in the target account.
Two constraints must hold or dashboards will silently lose data:

- **Schedules must run after the nightly Glue ETL lands data in Redshift** (currently ~06:10 UTC),
  and must be defined in the **UTC timezone** so daylight-saving transitions never shift them
  relative to Glue's UTC cron trigger.
- **The look-back window must be at least 2 days.** Events reach Redshift up to ~27 hours after
  their timestamps (the nightly batch covers the previous day). With a shorter window, rows land
  after every refresh window that covered their timestamps has passed, and they are permanently
  skipped — this caused a nightly 1–4am gap in all dashboards from May to August 2026.

Note that `export-analysis` regenerates these files from the *live* schedules of the account being
exported from, so keep the authoring account's schedules aligned with these values (or re-check the
diff after every export). 
