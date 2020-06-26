<a title="Frank Schulenburg / CC BY-SA (https://creativecommons.org/licenses/by-sa/3.0)" href="https://commons.wikimedia.org/wiki/File:Western_Grebe_(Aechmophorus_occidentalis)_IV.jpg"><img width="1024" alt="Western Grebe (Aechmophorus occidentalis) IV" src="https://upload.wikimedia.org/wikipedia/commons/thumb/5/55/Western_Grebe_%28Aechmophorus_occidentalis%29_IV.jpg/1024px-Western_Grebe_%28Aechmophorus_occidentalis%29_IV.jpg"></a>
# Determining Release Locations for Rehabilitated Birds

I am a volunteer for [International Bird Rescue](https://www.bird-rescue.org/), and one of
the amazing things I get to do as a volunteer (besides cleaning enclosures and cutting up fish) is release rehabilitated birds back into the wild. I am also interested in GIS, and for a geospatial analysis project I decided to look at the question of where to release birds that have been rehabilitated at the local IBR wildlife center.

I'm not a bird expert, so some of the assumptions that I made while developing this analysis will be wrong. It is an experiment, and my hope is that it can at least serve as as starting point for considering how GIS can be applied to this question and what changes are needed for it to be useful.

1. [Background](#background)
2. [Data Sources](#data-sources)
3. [Suitability Analysis](#suitability-analysis)
4. [Results](#results)
5. [Discussion](#discussion)

*This material was created for my capstone project for the [Coursera Specialization in GIS](https://www.coursera.org/specializations/gis) by UC Davis. The results do not represent the policies or practices of International Bird Rescue. I'm just a volunteer. All assumptions, interpretations, and errors are my own.*

## Background

Wildlife rehabilitation is the practice of treating and caring for sick, injured, or orphaned wild animals. The wildlife rehabilitator’s goal is to return the animal to its life in the wild once it is healthy and able to take care of itself. To give animals their best chance to thrive, the rehabilitator tries to release them in the right habitat where they will have access to the right food and shelter.

Release sites are chosen by the wildlife rehabilitator based on their experience and the advice of wildlife experts. This project explores using a data-driven approach to finding release locations by performing a suitability analysis using data about species abundance, protected wildlife area boundaries, and proximity to the rehabilitation center.

[International Bird Rescue](https://www.bird-rescue.org/) (IBR) specializes in rehabilitating aquatic birds, primarily at its wildlife centers in Los Angeles and the San Francisco Bay Area. IBR routinely handles a variety of bird species, so in order to give focus to this project, I will specifically look at the Western Grebe, IBR’s Bird of the Year for 2020 and one of the more commonly treated species. This project seeks to answer where IBR should release Western Grebes that have been rehabilitated at their San Francisco Bay-Delta Wildlife Center. The method developed can easily be applied to other bird species for which data is available.

## Data Sources

### Abundance Data

The most important factor when choosing a place to release a bird is habitat, which simply means where that kind of bird likes to live. An interesting source of data about habitat is available from [eBird](https://ebird.org/science) Status and Trends.[^1] They provide freely available abundance and range data for hundreds of birds species based on observations made by the global birding community.

*Abundance* is a relative measure of how many birds of a given species are estimated to be in a certain area during a certain part of the year.[^2] I use abundance as a simple proxy for habitat. We want to know where a bird is likely to thrive in the wild, and the more members of its species that can be found in an area, the better that area is likely to be.

I downloaded raster datasets representing abundance of Western Grebes by using the [ebirdst](https://cornelllabofornithology.github.io/ebirdst) package for [R](https://www.r-project.org/). The data consists of GeoTIFF rasters representing abundance by week, by season (breeding, post-breeding migration, non-breeding, pre-breeding migration), and a year-round average.

*Abundance of Western Grebe in the study area during its breeding season. Darker colors represent greater abundance. The lightest shade of blue represents zero abundance, meaning Western Grebes are not seen in that area.*

![small map of abundance](/assets/images/bird_release/abund_wesgre_br.png)

### Wildlife Area Boundaries

I wanted to give a bonus to locations in or near protected wildlife areas, such as National Wildlife Refuges (NWRs). This factor may not be important in practice and should be reevaluated.

US Fish & Wildlife Service (USFWS) boundary data was retrieved from [USFWS Geospatial Services](https://www.fws.gov/gis/data/CadastralDB/links_cadastral.html). This data set includes features showing the boundaries of NWRs.

California Department of Fish and Wildlife (CDFW) owned and operated lands data was retrieved from the [CDFW GIS Clearinghouse](https://wildlife.ca.gov/Data/GIS/Clearinghouse). This data includes features showing the boundaries of state wildlife areas and ecological reserves.

### Drive-Time Area

Travelling in a crate is stressful for a wild animal, so we would like to limit it to a reasonable amount of time. I factored this into the analysis by creating *drive-time area* polygon features (also known as *isochrones*) that represent the usual distance you could travel from the rehab center by car in a certain amount of time.

Drive-time area polygon features were generated using [openrouteservice](https://maps.openrouteservice.org/). Generating drive-time areas is relatively expensive, so I used just two: a 30-minute area and a 60-minute area.

![small drive-time areas map](/assets/images/bird_release/isochrones.png)

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

## Results

Here is a closer look at the breeding season release value map in which I have highlighted the highest-scoring areas. Red dots show actual sites where IBR volunteers often release Western Grebes.[^3]

![small map of top release values](/assets/images/bird_release/relval_wesgre_br_top.png)

**Areas closest to wildlife center:** these areas have medium abundance, are nearby, and include parts of the Grizzly Island Wildlife Area. Unfortunately, while some parts of the Grizzly Island Wildlife Area are open to the public, these particular parts do not appear to be actually accessible. Interestingly, IBR regularly releases Western Grebes in Benicia and Martinez, immediately to the southwest of these areas.

**Areas west of the wildlife center:** these areas have medium abundance, are at the limit of the 30-minute drive area, and include parts of the San Pablo Bay NWR and Napa-Sonoma Marshes Wildlife Area.

**Area southwest of the wildlife center:** this area has fairly high abundance and is near the Corte Madera Marsh Ecological Reserve.

**Areas north of the wildlife center:** these areas are at Lake Berryessa which has some of the highest abundance values in the region during the breeding season. There is a CDFW wildlife area nearby (Putah Creek Wildlife Area) but I'm not sure it contains any actual Western Grebe habitat.

## Discussion

This analysis can suggest general areas for release, but the selection of a specific site within that area will still require an experienced eye.

- The areas highlighted above received the highest scores in this analysis, but the score cutoff was arbitrarily selected (greater or equal to 80% of the maximum release value). What score is good enough to consider releasing in that area?

- One limitation of the analysis is the resolution of the original abundance data. The 3km resolution means that raster cells cover 9 square km (about 3.5 square miles). Exactly where within that area does it make sense to release?

- I tried to emphasize accessibility with the drive-time and wildlife areas, but there is no guarantee that a high-scoring area is actually accessible. A human familiar with the area needs to make the decision of whether it makes sense to go there. This issue is exacerbated by the low resolution of the analysis.

- IBR uses volunteers to release birds when possible. This can factor into the choice of release site because they try to choose a place close to where the volunteer lives, and a particular volunteer might live anywhere within the San Francisco Bay Area.

The upshot of these considerations is that the result of this analysis should be viewed as a suggestion or a point of reference in making the decision of where to release.

---

[^1]: [eBird Status and Trends](https://ebird.org/science/status-and-trends): Fink, D., T. Auer, A. Johnston, M. Strimas-Mackey, O. Robinson, S. Ligocki, B. Petersen, C. Wood, I. Davies, B. Sullivan, M. Iliff, S. Kelling. 2020. eBird Status and Trends, Data Version: 2018; Released: Cornell Lab of Ornithology, Ithaca, New York. https://doi.org/10.2173/ebirdst.2018

[^2]: [How is relative abundance defined?](https://ebird.org/science/status-and-trends/faq#mean-relative-abundance)

[^3]: A list of actual release sites was provided by Cheryl Reynolds at IBR.
