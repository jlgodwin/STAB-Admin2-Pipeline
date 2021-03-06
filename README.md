# STAB Admin 2 U5MR Pipeline

Pipeline code for development of subnational estimates of U5MR using [`SUMMER`](https://github.com/richardli/SUMMER).

Click here for the [**SPACE STATION**](http://faculty.washington.edu/jonno/space-station.html)

Click here for the [**COMMAND CENTER**](https://docs.google.com/spreadsheets/d/1GgrysoVHM2bO6DUZx8Cmj7WICKZ5KpTay0GOT72zK24/edit#gid=0)

![Pipeline Schematic](Figs/Pipeline.png) 

# Example data
  * Country-specific data
    * Togo 2013-14, 1998 DHS Births Recode (STATA)
    * Togo 2013-14, 1998 DHS Geographic Dataset (FLAT ASCII/reads in as `SpatialPointsDataFrame`)
    * Togo GADM Polygon files (reads in as `SpatialPolygonsDataFrame`)
  * Meta-data
    * SurveyNum.csv
      - Contains CountryName, unique DHS survey number, DHS survey name key
    * SurveyList_BR_GE.csv
      - Contains SO MUCH INFO including last upload to [DHS](https://www.dhsprogram.com), survey type, data format, filenames, etc. 
    * CountryList.csv or [**Command Center: Country List**](https://docs.google.com/spreadsheets/d/1GgrysoVHM2bO6DUZx8Cmj7WICKZ5KpTay0GOT72zK24/edit#gid=0)
    * SurveyInfo.csv or [**Command Center: Survey Info**](https://docs.google.com/spreadsheets/d/1GgrysoVHM2bO6DUZx8Cmj7WICKZ5KpTay0GOT72zK24/edit#gid=1656161984)

# .R Files

  1. **DataProcessing.R**
     *  Combines
        - DHS Births Recode files (\*.dta), 
        - DHS GPS files (\*.cpg, \*.dbf, \*.prj, \*.sbn, \*.sbx, \*.shp, \*.shp.xml, \*.shx),
        - GADM (usually) polygon files (\*.cpg, \*.dbf, \*.prj, \*.shp,  \*.shx) 
     *  Associated `SUMMER` functions: 
        - `getBirths()`: loads DHS Births Recode STATA file from filename and prepares data for survival analysis based on birth history data
          - Section: Load Data
     *  Products:
        - `admin1.mat`, `admin2.mat`: `matrix` describing neighborhood structure of subnational polygons for future use in `SUMMER` and nested `INLA` functions
           - Section: Create adjacency matrices
           - Output: data.dir/folder.name/shapes.sub.dir/CountryName_Amat.rda 
        - `admin1.names`, `admin2.names`: `data.frame` keys matching GADM area name strings to internal (unique) area name strings
          - Section: Create adjacency matrices 
          - Output: data.dir/folder.name/shapes.sub.dir/CountryName_Amat_Names.rda
        - `mod.dat`: `data.frame` that summarizes number of deaths and agemonths for each cluster and contains GPS location of clusters and Admin area assignments
          - Sections: Combine survey frames, Check frame, Save frame 
          - Output: data.dir/folder.name/CountryName_cluster_dat.rda
     *  Plots:
        - Creates directories: data.dir/folder.name/Plots, data.dir/folder.name/Plots/ShapeCheck
        - neighborhood structure on polygons
          - data.dir/folder.name/Plots/ShapeCheck/CountryName_adm\*\_neighb.pdf
        - cluster locations in DHS Geographic Data
          - data.dir/folder.name/Plots/ShapeCheck/Points_SurveyYear_GE.png
        - cluster locations from DHS Geographic that are present in Births Recode file 
          - data.dir/folder.names/Plots/ShapeCheck/Points_Survery_Year_BR.png 
  2. **DirectEstimates.R**
     *  Uses output of **DataProcessing.R** to get design-based discrete hazards estimates and SEs of child mortality adjusts for HIV when appropriate
     *  Associated `SUMMER` functions: 
        - `getDirectList()`: takes output of `getBirths()` and computes H-T estimates of U5MR
          - Section: Direct Estimates- National, Admin1, Admin2
        - `getDirect()`: takes output of `getBirths()` and computes H-T estimates of U5MR when country has single survey
          - Section: Direct Estimates- National, Admin1, Admin2
        - `getAdjusted()`: takes output of `getDirect*()` and divides estimates by a constant/adjsutst SEs; used for HIV adjustment and benchmarking for direct estimates
          - Section: Direct Estimates- HIV Adjustment
        - `mapPlot()`: takes output of `getDirect*()`, `getAdjusted()`, or `aggregateSurvey()` and plots specified column in that output for specified time period on provided `SpatialPolygonsDataFrame`
          - Section: Polygon Plots- National, Admin1, Admin2- By Survey
        - `aggregateSurvey()`:  takes output of `getDirectList()` to compute meta-analysis estimator for each area and time (i.e. aggregating across surveys)
          - Section: Polygon Plots- National, Admin1, Admin2- Meta-analysis
     * Products:
       - `direct.natl`: `data.frame` output from `getDirect()` or `getDirectList()` with national Horvitz-Thompson estimates of child mortality by 5-year period and survey
         - Section: Direct Estimates (National)
         - Output: data.dir/folder.name/CountryName_direct_natl.rda
       - `direct.natl.yearly`: `data.frame` output from `getDirect()` or `getDirectList()` with national Horvitz-Thompson estimates of child mortality by yearand survey
         - Sections: Direct Estimates (National), HIV Adjustments (if appropriate)
         - Output: data.dir/folder.name/CountryName_direct_natl_yearly.rda, data.dir/folder.name/CountryName_directHIV_natl_yearly.rda
       - `direct.admin1`: `data.frame` output from `getDirect()` or `getDirectList()` with Admin1 Horvitz-Thompson estimates of child mortality by  5-year period and survey
         - Sections: Direct Estimates (Admin1), HIV Adjustments (if appropriate)
         - Output: data.dir/folder.name/CountryName_direct_admin1.rda, data.dir/folder.name/CountryName_directHIV_admin1.rda
       - `direct.admin2`: `data.frame` output from `getDirect()` or `getDirectList()` with Admin2 Horvitz-Thompson estimates of child mortality by  5-year period and survey
         - Sections: Direct Estimates (Admin2), HIV Adjustments (if appropriate)
         - Output: data.dir/folder.name/CountryName_direct_admin2.rda, data.dir/folder.name/CountryName_directHIV_admin2.rda
     * Plots:
       - Creates directory: folder.name/Plots/Direct
       -  Polygon plots direct estimates by administrative division
          - **By survey:** data.dir/folder.name/Plots/Direct/\*\/CountryName_\*\_direct_poly_BySurvey_Period.pdf
            -   \*: National, Admin1, Admin2 in the file path and natl, admin1, admin2 in the file name
          - **Meta-analysis estimator:** data.dir/folder.name/Plots/Direct/\*\/CountryName_\*\_direct_poly_Meta.pdf
            -   \*: National, Admin1, Admin2 in the file path and natl, admin1, admin2 in the file name
          - if a country requires HIV adjustment, the plots display the adjusted estimates
       - Spaghetti plots of direct estimates by administrative division
         - data.dir/folder.name/Plots/Direct/\*\/CountryName_\*\_direct_spaghetti.png
            -  \*: National, Admin1, Admin2 in the file path and natl, admin1, admin2 in the file name
         -  (for some reason) data.dir/folder.name/Plots/Direct/National/CountryName_natl_direct_yearly_spaghetti.png
         - if a country requires HIV adjustment, the plots display the adjusted estimates
  3. **SmoothedDirect.R**
      * Uses output of **DataProcessing.R** and **DirectEstimates.R** to get estimates of child mortality smoothed in space and time, benchmarks to UN IGME when `doBenchmark == TRUE`
      * Associated `SUMMER` functions:
        - `aggregateSurvey()`: takes output of `getDirectList()` to compute meta-analysis estimator for each area and time (i.e. aggregating across surveys)
          - Section:  Aggregate surveys
        - `smoothDirect()`: takes output of `aggregateSurvey()` (or `getDirectList()`) and fits area-level spatiotemporal smoothing model in `INLA`
          - Sections: Fit smoothing model, Benchmarking
        - `getSmoothed()`: takes output of `smoothDirect()` to get spatiotemporal estimates of child mortality in easily usable format
          - Sections: Fit smoothing model, Benchmarking
        - `getAdjusted()`: takes output of `aggregateSurvey()` (or `getDirectList()`) and benchmarks to UN IGME estimates
          - Sections: Benchmarking
        - `mapPlot()': takes output of `getSmoothed()` and plots median estimates of U5MR from smoothed direct model by 5-year period
          - Sections: Polygon plots  
      * Products:
        - `res.natl`
           - Section: Fit smoothing model (National), Benchmarking (National)
           - Output: data.dir/folder.name/CountryName_res_natl_SmoothedDirect.rda, data.dir/folder.name/CountryName_res_natlBench_SmoothedDirect.rda
        - `res.natl.yearly`
           - Section: Fit smoothing model (National), Benchmarking (National Yearly)
           - Output: data.dir/folder.name/CountryName_res_natl_yearly_SmoothedDirect.rda, data.dir/folder.name/CountryName_res_natlBench_yearly_SmoothedDirect.rda
        - `res.admin1`
           - Section: Fit smoothing model (Admin1), Benchmarking (Admin1)
           - Output: data.dir/folder.name/CountryName_res_admin1_SmoothedDirect.rda, data.dir/folder.name/CountryName_res_admin1Bench_SmoothedDirect.rda
       * Plots:
         - Creates directory: data.dir/folder.name/Plots/SmoothedDirect
         - `SUMMER` plots using `plot`, plots benchmarked results if `doBenchmark == TRUE`
            - data.dir/folder.name/Plots/SmoothedDirect/\*\/CountryName_\*\_SmoothedDirect.pdf
              - \*: National, Admin1, Admin2 in the file path and natl, admin1, admin2 in the file name 
         - Spaghetti Plots
            - data.dir/folder.name/Plots/SmoothedDirect/\*\/CountryName_\*\_SmoothedDirect_spaghetti\*\.pdf
              - \*: National, Admin1, Admin2 in the file path and natl, admin1, admin2 in the file name 
         - Polygon Plots with `SUMMER::mapPlot`
            - data.dir/folder.name/Plots/SmoothedDirect/Admin1/CountryName_Admin1_SmoothedDirect_poly.pdf 
  5. **Betabinomial.R**  
     * Uses output of **DataProcessing.R** to get model-based discrete hazards estimates of child mortality smoothed in space and time, adjusts for HIV when appropriate and benchmarks to UN IGME when `doBenchmark == TRUE`
     * `doNatl` logical for fitting RW2 national model
     * `doAdmin1` or `doAdmin2` logical for fitting either Admin1 or Admin2 level model with RW2 main temporal effects
     * `doRandomSlopesRW1` or `doRandomSlopesAR1` for fitting the respective temporal model in the Knorr-Held Type III or Type IV
     * `refit` refits the unbenchmarked model
     * `refitBench` refits benchmarked model
     * `loadSamples` loads last posterior draws (should be updated to be `loadSamples` and `loadSamplesBench`)
     * Associated `SUMMER` functions:
       - `smoothCluster()`: takes output of `getBirths()` and fits spatiotemporal unit-level model in `INLA` and returns a list of objects related to model estimation
       - `getDiag()`: takes output of `smoothCluster()` and returns smaller object of latent terms determined by passing specifying `field = "space"`, `field = "time"`, or `field = "spacetime"`
       - `getSmoothed()`: takes output of `smoothCluster()` and returns a list of desired results and posterior summaries of the estimated model including posterior summaries of U5MR by area and year and draws from the joint posterior distribution of model parameters, and the transformation of those draws into U5MR by area and year
     * Products:
       - 
