---
layout: post
title: Release Locations for Rehabilitated Birds - Technical Details
---
These are low-level details of the suitability analysis I performed for [rehabilitated bird release locations](/gis/bird_release).

## Data Sources

First, a quick recap of the data sources I used.

### Abundance Data

I downloaded raster datasets representing abundance of Western Grebes by using the [ebirdst](https://cornelllabofornithology.github.io/ebirdst) package for [R](https://www.r-project.org/). The data consists of GeoTIFF rasters representing abundance by week, by season (breeding, post-breeding migration, non-breeding, pre-breeding migration), and a year-round average.

### Wildlife Area Boundaries

US Fish & Wildlife Service (USFWS) boundary data was retrieved from [USFWS Geospatial Services](https://www.fws.gov/gis/data/CadastralDB/links_cadastral.html). This data set includes features showing the boundaries of NWRs.

California Department of Fish and Wildlife (CDFW) owned and operated lands data was retrieved from the [CDFW GIS Clearinghouse](https://wildlife.ca.gov/Data/GIS/Clearinghouse). This data includes features showing the boundaries of state wildlife areas and ecological reserves.

### Drive-Time Areas

Drive-time area polygon features were generated using [openrouteservice](https://maps.openrouteservice.org/). We don't need a lot of precision here, so I used just two: a 30-minute area and a 60-minute area. Openrouteservice exports the features as GeoJSON, and I used QGIS to convert them to a shapefile for use in ArcMap.

## Suitability Analysis

Suitability was determined by assigning scores to the various data inputs and then adding the score rasters together to get a final score.

- Abundance
    1. The eBird abundance rasters use a global sinusoidal projection, so I projected them to the NAD 1983 California (Teale) Albers projection, with extent limited to the search area. The resolution of the source data is approximately 3km x 3km, and this cell size was used for all subsequent raster processing in this analysis.

        *This is what the abundance data looks like before it is re-projected. California is very distorted in the sinusoidal projection.*

        ![unprojected abundance](/assets/images/bird_release/abund_wesgre_br_unproj.png)

    2. The projected abundance raster was masked to the search area, which was defined as the area within an 85km radius of the wildlife center. A small area (3km radius around the center) was excluded to represent the fact that we generally avoid releasing in the immediate vicinity of the center itself.

        ![small map of abundance](/assets/images/bird_release/abund_wesgre_br.png)

    3. All cells below some threshold (0.1) were changed to NoData. If the abundance value in a cell is very low, that species is rarely or never seen in that area so we exclude it from the results.

    4. The abundance raster was reclassified using a linear scaling function such that the maximum abundance value within the search area got a score of 10. *Linear* means that scores are in proportion to abundance; if area A has half the abundance of area B, then it will receive half the score.

        *Abundance score for the Western Grebe during its breeding season. Darker cells have a higher score.*

        ![small map of abundance score raster](/assets/images/bird_release/abscore_wesgre_br.png)

- Wildlife Areas
    1. Features from the USFWS lands layer with a type (`RSL_TYPE`) of “NWR” and features from the CDFW lands layer with a type (`PROP_TYPE`) of “Wildlife Area” or “Ecological Reserve” and open accessibility (`ACCESS LIKE 'Open%'`) were merged into a new feature class representing all protected wildlife areas.

        *Green areas are NWRs, orange areas are CDFW lands.*

        ![small map of wildlife areas](/assets/images/bird_release/wildlifeareas.png)

    2. The protected wildlife areas were buffered by 1.5km and then converted to a raster with cell size and alignment matching the abundance raster. Buffering by 1.5 km is a simple way of improving the coverage of cells when converting to the 3km resolution raster, so that most of the cells that intersect one of the original features will be included.
    ![small map of buffered wildlife areas](/assets/images/bird_release/wildlifeareas_buf.png)
    3. The protected wildlife area raster was reclassified by giving cells that touch a wildlife area a score of 3 and all other cells a score of zero.
    ![small map of wildlife area raster](/assets/images/bird_release/wildlifeareas_raster.png)

- Drive-time Area
    1. The drive-time areas were buffered by 1.5 km and then converted into a raster with cell size and alignment matching the abundance raster.

        ![drive-time area raster](/assets/images/bird_release/isochrones_raster.png)

    2. The drive-time raster was reclassified for suitability score by giving cells in the 30 minute region a score of 5, cells in the 60 minute region but outside the 30 minute region a score of 2, and all other cells a score of zero.

The score rasters were added together to create the final suitability raster. I refer to the final score for an area as its *release value*.

*Western Grebe release value, breeding season (May 31 - Jul 20). Yellow values are lowest, dark blue are highest.*

![small release value map](/assets/images/bird_release/relval_wesgre_br.png)
