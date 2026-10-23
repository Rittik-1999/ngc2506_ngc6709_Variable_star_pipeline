# Variable-star pipeline: NGC 2506 and NGC 6709

`ngc6709.ipynb` contains the photometry, candidate discovery, period analysis, Gaia DR3 cross-match, all-star astrometric membership and colour–magnitude analysis for both clusters. Input observations are supplied separately. The notebook does not include manuscript-selected star lists, isochrone comparisons, final-table copying or personal figure-assembly steps.

## Inputs and setup

Use Python with Jupyter, NumPy, pandas, SciPy, Astropy, astroquery and Matplotlib installed in the same environment. An internet connection is needed for a fresh Gaia query.

The setup cell defaults to this layout, relative to the notebook working directory:

```text
data/
  NGC2506/RFILTER/
    JD_all.dat
    ref_stars_xy_radec.dat
    refimage.ref
    <science-frame>.ref
  NGC6709/RFILTER/
    JD_all.dat
    ref_stars_xy_radec.dat
    refimage.ref
    <science-frame>.ref
outputs/
```

Set `DATA_ROOT`, `OUTPUT_ROOT` and `SELECTED_CLUSTERS` in the setup cell. The default selection is both `ngc2506` and `ngc6709`. The cluster paths can also be edited individually. Outputs are separate from input data. Run from a working directory where creating the configured output directory is appropriate.

Input formats:

- `ref_stars_xy_radec.dat`: whitespace-separated `star_id x y RA Dec`, without a header. RA is sexagesimal hours and Dec is sexagesimal degrees. IDs identify the same physical stars throughout the cluster’s reference catalogue, candidate CSV and period outputs.
- `JD_all.dat`: frame filename and Julian date. Use times matching the declared input scale. The discovery reader expects filename first, JD second; science-frame names must resolve to the corresponding `.ref` basenames.
- `refimage.ref` and science `.ref` files: the DAOPHOT format used by the analysis, with three header rows followed by nine numeric columns. The first five columns are ID, x, y, magnitude and magnitude uncertainty. Positions must be in the common reference system closely enough for the configured matching/affine-seed tolerances.

The default input scale is UTC. Period analysis applies heliocentric light-travel-time correction and uses **HJD on the UTC scale**. `TIME_BASIS_SETTING` and `JD_INPUT_SCALE_SETTING` make those assumptions explicit. The observatory coordinates and analysis thresholds are defined in the period cells.

## Run order

Run all sections from the top:

1. **Setup:** paths, cluster selection, time basis and Gaia cache preference.
2. **Candidate discovery:** run the same discovery calculation separately for each selected cluster. This writes its own candidate/comparison/check-star list.
3. **NGC 2506 periods:** read that cluster’s candidate list and raw frames. This section runs only when NGC 2506 is selected.
4. **NGC 6709 periods:** likewise for NGC 6709, using the supplied first comparison star.
5. **Gaia cross-match:** process every selected cluster, using the nearest Gaia DR3 source within the configured match radius.
6. **Membership:** fit the all-star astrometric model for each selected cluster.
7. **CMD:** read the all-star membership output, apply adopted reddening/distance settings and save the corrected CMD.

Keep each period engine’s definitions and execution together in its cell: the two engines have overlapping helper names. To rerun either analysis, rerun its complete cell after setup. Raw photometry is rebuilt each time; an old photometry cache is not used. Full frame processing and repeated period searches can take time and memory.

Gaia caching is optional and disabled by default. With `USE_GAIA_CACHE=True`, cone-query snapshots are stored under `outputs/cache/`. Their filenames include an identity derived from the reference-catalogue contents and query/matching controls. Cached source IDs are read as strings to preserve their digits. Delete the relevant snapshot or disable caching to request fresh archive data.

## Outputs

`<cluster>` below means `ngc2506` or `ngc6709`.

| Location under `OUTPUT_ROOT` | Contents |
| --- | --- |
| `<cluster>/cell0_all_star_diagnostics.csv` | Per-star discovery measurements |
| `<cluster>/cell1_variable_candidates.csv` | Ranked candidates, comparison/check IDs and discovery diagnostics |
| `ngc2506/periods/04_period_phase_summary.csv` | Adopted NGC 2506 periods and search diagnostics |
| `ngc2506/periods/04_period_phase_top_candidates.csv` | Alternative period candidates |
| `ngc2506/periods/04_relative_lightcurves_points_newid.csv` | Analysed differential light-curve points, errors and adopted periods |
| `ngc2506/periods/04_period_phase_frame_diagnostics.csv` | Frame matching and zero-point diagnostics |
| `ngc2506/periods/periodogram_phase_plots/04_variable_star_amplitude_skewness_period.csv` | Period, amplitude, shape/Fourier features and conditional uncertainties |
| `ngc6709/cell2_period_phase_summary.csv` | Periods, statuses, shape/Fourier features and conditional uncertainties |
| `ngc6709/cell2_phase_quality_ranking.csv` | Accepted results ranked by phase-quality diagnostics |
| `<cluster>/...` | Periodogram/phase figures in the engine’s plot subdirectory |
| `<cluster>/<cluster>_gaia_crossmatch.csv` | Gaia match, astrometry, photometry and match separation |
| `tables/<cluster>_membership_all_stars.csv` | All-star PM-only probability, joint astrometric probability, gated score and membership labels |
| `tables/<cluster>_proper_motion_radial_density.csv` | Observed/fitted annular PM-density profile |
| `tables/membership_parameters.csv` | Fitted parameters and all-star membership counts |
| `tables/<cluster>_cmd_all_stars.csv` | All-star membership catalogue with adopted extinction/distance and corrected CMD columns |
| `figures/<cluster>/` | Membership and CMD figures, as PDF and PNG |

These are upstream scientific products for a later ML workflow; this notebook does not assemble a selected nine-feature ML input table. Preserve star IDs and Gaia source IDs exactly when joining products. Review result statuses before selecting stars. The two period engines use different amplitude/skewness estimators, so a later classifier must explicitly reconcile its training and inference feature definitions.

## Method and interpretation

Candidate ranking uses binned differential variability, not a calibrated detection probability. The period engines retain their configured alias/harmonic rules. In NGC 6709, explicit long/short-period preference terms contribute to selection; the search is not distribution-neutral. Those settings are exposed in the code and are not retuned by this release.

Phase figures show selected real observations near local median trends. Their apparent scatter does not represent all analysed measurements. The period/amplitude uncertainty calculation uses session-aware/residual resampling and local frequency perturbations; period errors are conditional on the selected alias and do not represent alternative daily aliases or half/double periods. NGC 2506’s optional feature-stabilisation step can evaluate shape features at a nearby period while retaining the adopted period in the summary.

Membership uses fixed proper-motion centres and parallax gates, a fitted core-plus-wing PM-density model, and a joint parallax likelihood. `p_mu`, `p_ast` and `gated_astrometric_membership_score` have distinct meanings. Secure/borderline labels follow the configured gates and thresholds; no manuscript-selected IDs enter the fits.

The CMD uses one adopted reddening and true distance modulus per cluster. Corrected magnitudes at the common cluster distance support sequence comparisons; they are not individual distance estimates for field stars. The separate Gaia cross-match also retains direct positive-parallax absolute-magnitude estimates.

## License

This project is distributed under the BSD 3-Clause License.
See [LICENSE](LICENSE) for details.

## Archived release

Version 0.1.0 is archived on Zenodo:
[View the archived release](https://doi.org/10.5281/zenodo.23596542).
