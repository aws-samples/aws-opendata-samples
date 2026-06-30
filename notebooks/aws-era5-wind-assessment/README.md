# Wind Energy Potential Assessment using the ERA5-WRF Dataset

This notebook demonstrates a practical use case: assessing **wind energy potential** for a
location using the ERA5-WRF global timeseries data available on the AWS Open Data Registry.

## Use Case

Wind farm developers and energy planners need to understand wind patterns at potential sites.
This analysis:

- Downloads historical wind data for a location directly from a public S3 bucket
- Analyzes the wind speed distribution (Weibull fit) and temporal patterns
- Estimates energy production potential with an air-density correction
- Visualizes wind direction frequency and energy by sector

The notebook uses a location in **West Texas** (near Sweetwater, a major wind farm area) as a
worked example, but the target latitude/longitude can be changed to analyze any location.

## Dataset

The notebook uses the **Global Timeseries** product from ERA5-WRF, served from the public
`era5-for-wrf` S3 bucket (no AWS credentials required). It provides:

- 100-m wind speed and direction (relevant for modern wind turbines)
- Hourly resolution from 2000 to present
- 0.25° global grid (~28 km resolution)

See the dataset on the AWS Open Data Registry for more details.

## Files

- **`wind_energy_assessment_era5.ipynb`** - the wind energy assessment notebook
- **`environment.yml`** - conda environment with the packages needed to run the notebook

## Running the Notebook

The notebook reads from a public S3 bucket using unsigned requests, so no AWS account or
credentials are required.

### Option 1: Local / conda

```bash
conda env create -f environment.yml
conda activate aws_era5_wind_assessment
jupyter notebook wind_energy_assessment_era5.ipynb
```

### Option 2: SageMaker Studio Lab

Upload this folder and build the conda environment from `environment.yml`, then open
`wind_energy_assessment_era5.ipynb`.

The required packages can also be installed directly with pip:

```bash
pip install boto3 pandas numpy matplotlib scipy -q
```

## Security

See [CONTRIBUTING](CONTRIBUTING.md#security-issue-notifications) for more information.

## License

This library is licensed under the MIT-0 License. See the LICENSE file.

## Code of Conduct

This project has adopted the [Amazon Open Source Code of Conduct](https://aws.github.io/code-of-conduct).
For more information see the [Code of Conduct FAQ](https://aws.github.io/code-of-conduct-faq) or contact
opensource-codeofconduct@amazon.com with any additional questions or comments.
