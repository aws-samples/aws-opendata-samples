# Wind Energy Potential Assessment using ERA5 Datasets

These notebooks demonstrate a practical use case: assessing **wind energy potential** using
ERA5 reanalysis data available on the AWS Open Data Registry.

- **`wind_energy_assessment_era5.ipynb`** - single-location assessment using the ERA5-WRF
  global timeseries data (`era5-for-wrf` bucket)
- **`wind_energy_assessment_ncar_era5.ipynb`** - regional wind resource mapping using the
  NSF NCAR Curated ERA5 dataset (`nsf-ncar-era5` bucket)

## Use Case

Wind farm developers and energy planners need to understand wind patterns at potential sites.

**`wind_energy_assessment_era5.ipynb`** focuses on a single location:

- Downloads historical wind data for a location directly from a public S3 bucket
- Analyzes the wind speed distribution (Weibull fit) and temporal patterns
- Estimates energy production potential with an air-density correction
- Visualizes wind direction frequency and energy by sector

It uses a location in **West Texas** (near Sweetwater, a major wind farm area) as a worked
example, but the target latitude/longitude can be changed to analyze any location.

**`wind_energy_assessment_ncar_era5.ipynb`** focuses on a region:

- Accesses monthly NetCDF4 files anonymously and selects a geographic region before loading
  data into memory
- Derives mean wind speed and reference-density wind power density across the region
- Calculates an illustrative normalized output factor with a generic power curve
- Compares the highest-resource grid cells for the selected period

It uses 100-m wind fields over **Texas and the US South-Central region** as a worked example.

## Datasets

Both notebooks read from public S3 buckets, so no AWS credentials are required.

- **ERA5-WRF Global Timeseries** (`era5-for-wrf` bucket): 100-m wind speed and direction,
  hourly resolution from 2000 to present, 0.25° global grid (~28 km resolution)
- **NSF NCAR Curated ERA5** (`nsf-ncar-era5` bucket): monthly global NetCDF4 surface-analysis
  files with 100-m U/V wind components on the same 0.25° grid

See the datasets on the AWS Open Data Registry for more details.

## Files

- **`wind_energy_assessment_era5.ipynb`** - single-location wind energy assessment notebook
- **`wind_energy_assessment_ncar_era5.ipynb`** - regional wind resource mapping notebook
- **`environment.yml`** - conda environment with the packages needed to run both notebooks

## Running the Notebooks

The notebooks read from public S3 buckets using unsigned requests, so no AWS account or
credentials are required.

### Option 1: Local / conda

```bash
conda env create -f environment.yml
conda activate aws_era5_wind_assessment
jupyter notebook
```

Then open either notebook.

### Option 2: SageMaker Studio Lab

Upload this folder and build the conda environment from `environment.yml`, then open the
notebook of your choice.

The required packages can also be installed directly with pip:

```bash
pip install boto3 pandas numpy matplotlib scipy s3fs xarray h5netcdf h5py cartopy global-land-mask -q
```

## Security

See [CONTRIBUTING](CONTRIBUTING.md#security-issue-notifications) for more information.

## License

This library is licensed under the MIT-0 License. See the LICENSE file.

## Code of Conduct

This project has adopted the [Amazon Open Source Code of Conduct](https://aws.github.io/code-of-conduct).
For more information see the [Code of Conduct FAQ](https://aws.github.io/code-of-conduct-faq) or contact
opensource-codeofconduct@amazon.com with any additional questions or comments.
