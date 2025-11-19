# housing-demand
//proportionate demand
let
    Source = Table.NestedJoin(#"Demand Export", {"Postcode PreFix"}, #"Client Region to Postcode", {"Post Code"}, "Client Region to Postcode", JoinKind.LeftOuter),
    #"Expanded Client Region to Postcode" = Table.ExpandTableColumn(Source, "Client Region to Postcode", {"Client Region"}, {"Client Region to Postcode.Client Region"})
in
    #"Expanded Client Region to Postcode"


    //AOP expanded 
    let
    Source = Csv.Document(File.Contents("C:\Users\emmahunter\Desktop\6a719b4e-2cd3-46a3-bd39-ba7a923a2986.csv"),[Delimiter=",", Columns=60, Encoding=1252, QuoteStyle=QuoteStyle.None]),
    #"Promoted Headers" = Table.PromoteHeaders(Source, [PromoteAllScalars=true]),
    #"Changed Type" = Table.TransformColumnTypes(#"Promoted Headers",{{"Registration date", type date}, {"Date form received from applicant", type date}, {"Received Date", type date}, {"HousingRegisterRef", type text}, {"Name", type text}, {"Date Of Birth", type date}, {"Age of household member Group", type text}, {"Age of household member", Int64.Type}, {"Correspondence Address1", type text}, {"Correspondence Address2", type text}, {"Correspondence Address3", type text}, {"Correspondence Address4", type text}, {"Correspondence Address5", type text}, {"Correspondence PostCode", type text}, {"Current Property Type", type text}, {"Tenancy Type", type text}, {"Household type", type text}, {"Current Tenure Type", type text}, {"Current Address 1", type text}, {"Current Address 2", type text}, {"Current Address 3", type text}, {"Current Address 4", type text}, {"Current Address 5", type text}, {"Current Postcode", type text}, {"Region", type text}, {"Number of people being rehoused", Int64.Type}, {"BandID", type text}, {"Strategic Bedroom Need", Int64.Type}, {"Points", Int64.Type}, {"Points Override", Int64.Type}, {"Points Calculated", Int64.Type}, {"Area of preference", type text}, {"Number of areas of preference", Int64.Type}, {"Does this applicant require a significantly adapted property as recommended by the OT?", type text}, {"Do you want or need any of the following types of accommodation? Housing with access for wheelchairs", type text}, {"Do you want or need any of the following types of accommodation? Sheltered housing", type text}, {"Do you want or need any of the following types of accommodation? Amenity Housing", type text}, {"Type of property Maisonette (upper floor)", type text}, {"Type of property Tenement (ground floor)", type text}, {"Type of property Four in a block (upper floor)", type text}, {"Type of property Maisonette (ground floor)", type text}, {"Type of property Three-storey end-terrace", type text}, {"Type of property Hostel", type text}, {"Type of property Tenement (upper floor)", type text}, {"Type of property Basement flat", type text}, {"Type of property Three-storey mid-terrace", type text}, {"Type of property Semi-detached house", type text}, {"Type of property Mid-terrace house", type text}, {"Type of property End-terrace house", type text}, {"Type of property Multi-storey flat", type text}, {"Type of property Four in a block (ground floor)", type text}, {"Type of property Detached house", type text}, {"Type of property Bungalow", type text}, {"Size of property Three bedrooms", type text}, {"Size of property Four bedrooms", type text}, {"Size of property Five or more bedrooms", type text}, {"Size of property Bedsit (no bedrooms)", type text}, {"Size of property One bedroom", type text}, {"Size of property Two bedrooms", type text}, {"", type text}})
in
    #"Changed Type"

    //raw data set orginal modified 

    let
    // Load audit data from CSV
    Source = Csv.Document(
        File.Contents("I:\HSPUBLIC\HQ Filing\Business Improvement\NAHR Demand\Demand Exports\Demand Export.csv"),
        [Delimiter=",", Encoding=1252, QuoteStyle=QuoteStyle.None]
    ),
    #"Promoted Headers" = Table.PromoteHeaders(Source, [PromoteAllScalars=true]),

    // Change column types
    #"Changed Type" = Table.TransformColumnTypes(#"Promoted Headers", {
        {"Registration date", type date}, {"Date form received from applicant", type date}, {"Received Date", type date},
        {"HousingRegisterRef", type text}, {"Name", type text}, {"Date Of Birth", type date},
        {"Age of household member Group", type text}, {"Age of household member", Int64.Type},
        {"Correspondence Address1", type text}, {"Correspondence Address2", type text}, {"Correspondence Address3", type text},
        {"Correspondence Address4", type text}, {"Correspondence Address5", type text}, {"Correspondence PostCode", type text},
        {"Current Property Type", type text}, {"Tenancy Type", type text}, {"Household type", type text},
        {"Current Tenure Type", type text}, {"Current Address 1", type text}, {"Current Address 2", type text},
        {"Current Address 3", type text}, {"Current Address 4", type text}, {"Current Address 5", type text},
        {"Current Postcode", type text}, {"Region", type text}, {"Number of people being rehoused", Int64.Type},
        {"BandID", type text}, {"Strategic Bedroom Need", Int64.Type}, {"Points", Int64.Type},
        {"Points Override", Int64.Type}, {"Points Calculated", Int64.Type}, {"Area of preference", type text},
        {"Number of areas of preference", Int64.Type},
        {"Does this applicant require a significantly adapted property as recommended by the OT?", type text},
        {"Do you want or need any of the following types of accommodation? Housing with access for wheelchairs", type text},
        {"Do you want or need any of the following types of accommodation? Sheltered housing", type text},
        {"Do you want or need any of the following types of accommodation? Amenity Housing", type text},
        {"Type of property Maisonette (upper floor)", type text}, {"Type of property Tenement (ground floor)", type text},
        {"Type of property Four in a block (upper floor)", type text}, {"Type of property Maisonette (ground floor)", type text},
        {"Type of property Three-storey end-terrace", type text}, {"Type of property Hostel", type text},
        {"Type of property Tenement (upper floor)", type text}, {"Type of property Basement flat", type text},
        {"Type of property Three-storey mid-terrace", type text}, {"Type of property Semi-detached house", type text},
        {"Type of property Mid-terrace house", type text}, {"Type of property End-terrace house", type text},
        {"Type of property Multi-storey flat", type text}, {"Type of property Four in a block (ground floor)", type text},
        {"Type of property Detached house", type text}, {"Type of property Bungalow", type text},
        {"Size of property Three bedrooms", type text}, {"Size of property Four bedrooms", type text},
        {"Size of property Five or more bedrooms", type text}, {"Size of property Bedsit (no bedrooms)", type text},
        {"Size of property One bedroom", type text}, {"Size of property Two bedrooms", type text}
    }),

    // Remove duplicates
    #"Removed Duplicates" = Table.Distinct(#"Changed Type", {"HousingRegisterRef"}),

    // Uppercase Postcode
    #"Uppercase Postcode" = Table.TransformColumns(#"Removed Duplicates", {{"Correspondence PostCode", Text.Upper, type text}}),

    // Add Postcode Prefix
    #"Added Postcode Prefix" = Table.AddColumn(#"Uppercase Postcode", "Postcode PreFix", each 
        Text.Upper(Text.Start(Text.Select([Correspondence PostCode], {"A".."Z","0".."9"}), 4)), type text),

    // Add Band1 Points Override
    #"Added Band1 Points Override" = Table.AddColumn(#"Added Postcode Prefix", "Band1 Points Override", each 
        if [BandID] = "Band 1" then -5 else [Points Override], Int64.Type),

// Add combined property options column with default "General Needs"
#"Added Property Options" = Table.AddColumn(#"Added Band1 Points Override", "Property Options", each
    let
        OT = if [#"Does this applicant require a significantly adapted property as recommended by the OT?"] = "Yes" then "OT adapted property" else null,
        Wheelchair = if [#"Do you want or need any of the following types of accommodation? Housing with access for wheelchairs"] = "Yes" then "Wheelchair" else null,
        Sheltered = if [#"Do you want or need any of the following types of accommodation? Sheltered housing"] = "Yes" then "Sheltered" else null,
        Amenity = if [#"Do you want or need any of the following types of accommodation? Amenity Housing"] = "Yes" then "Amenity" else null,
        OptionsList = List.RemoveNulls({OT, Wheelchair, Sheltered, Amenity}),
        Combined = if List.Count(OptionsList) = 0 then "General Needs" else Text.Combine(OptionsList, ", ")
    in
        Combined
),

    // Add combined Property Type
    #"Added Property Type" = Table.AddColumn(#"Added Property Options", "Property Type", each
        let
            Types = List.RemoveNulls({
                if [#"Type of property Maisonette (upper floor)"] = "Yes" then "Maisonette (Upper Floor)" else null,
                if [#"Type of property Tenement (ground floor)"] = "Yes" then "Tenement (Ground Floor)" else null,
                if [#"Type of property Four in a block (upper floor)"] = "Yes" then "Four in a Block (Upper Floor)" else null,
                if [#"Type of property Maisonette (ground floor)"] = "Yes" then "Maisonette (Ground Floor)" else null,
                if [#"Type of property Three-storey end-terrace"] = "Yes" then "Three-storey End-Terrace" else null,
                if [#"Type of property Hostel"] = "Yes" then "Hostel" else null,
                if [#"Type of property Tenement (upper floor)"] = "Yes" then "Tenement (Upper Floor)" else null,
                if [#"Type of property Basement flat"] = "Yes" then "Basement Flat" else null,
                if [#"Type of property Three-storey mid-terrace"] = "Yes" then "Three-storey Mid-Terrace" else null,
                if [#"Type of property Semi-detached house"] = "Yes" then "Semi-detached House" else null,
                if [#"Type of property Mid-terrace house"] = "Yes" then "Mid-terrace House" else null,
                if [#"Type of property End-terrace house"] = "Yes" then "End-terrace House" else null,
                if [#"Type of property Multi-storey flat"] = "Yes" then "Multi-storey Flat" else null,
                if [#"Type of property Four in a block (ground floor)"] = "Yes" then "Four in a Block (Ground Floor)" else null,
                if [#"Type of property Detached house"] = "Yes" then "Detached House" else null,
                if [#"Type of property Bungalow"] = "Yes" then "Bungalow" else null
            }),
            Combined = Text.Combine(Types, ", ")
        in
            Combined
    ),

    // Add combined Property Size
    #"Added Property Size" = Table.AddColumn(#"Added Property Type", "Property Size", each
        let
            Sizes = List.RemoveNulls({
                if [#"Size of property Bedsit (no bedrooms)"] = "Yes" then "Bedsit" else null,
                if [#"Size of property One bedroom"] = "Yes" then "1 Bed" else null,
                if [#"Size of property Two bedrooms"] = "Yes" then "2 Bed" else null,
                if [#"Size of property Three bedrooms"] = "Yes" then "3 Bed" else null,
                if [#"Size of property Four bedrooms"] = "Yes" then "4 Bed" else null,
                if [#"Size of property Five or more bedrooms"] = "Yes" then "5+ Bed" else null
            }),
            Combined = Text.Combine(Sizes, ", ")
        in
            Combined
    ),
   
#"Added Proportion" = Table.AddColumn(#"Added Property Size", "Proportion", each 
        if [Number of areas of preference] <> null and [Number of areas of preference] > 0 
        then 1 / [Number of areas of preference] 
        else null
    )


//AOP_VALUES_UPDATED
let
    // Load AOP expanded data from CSV
    Source = Csv.Document(
        File.Contents("C:\Users\emmahunter\Desktop\6a719b4e-2cd3-46a3-bd39-ba7a923a2986.csv"),
        [Delimiter=",", Columns=60, Encoding=1252, QuoteStyle=QuoteStyle.None]
    ),

    #"Promoted Headers" = Table.PromoteHeaders(Source, [PromoteAllScalars = true]),

    // Change column types
    #"Changed Type" = Table.TransformColumnTypes(#"Promoted Headers",{
        {"Registration date", type date}, 
        {"Date form received from applicant", type date}, 
        {"Received Date", type date}, 
        {"HousingRegisterRef", type text}, 
        {"Name", type text}, 
        {"Date Of Birth", type date}, 
        {"Age of household member Group", type text}, 
        {"Age of household member", Int64.Type}, 
        {"Correspondence Address1", type text}, 
        {"Correspondence Address2", type text}, 
        {"Correspondence Address3", type text}, 
        {"Correspondence Address4", type text}, 
        {"Correspondence Address5", type text}, 
        {"Correspondence PostCode", type text}, 
        {"Current Property Type", type text}, 
        {"Tenancy Type", type text}, 
        {"Household type", type text}, 
        {"Current Tenure Type", type text}, 
        {"Current Address 1", type text}, 
        {"Current Address 2", type text}, 
        {"Current Address 3", type text}, 
        {"Current Address 4", type text}, 
        {"Current Address 5", type text}, 
        {"Current Postcode", type text}, 
        {"Region", type text}, 
        {"Number of people being rehoused", Int64.Type}, 
        {"BandID", type text}, 
        {"Strategic Bedroom Need", Int64.Type}, 
        {"Points", Int64.Type}, 
        {"Points Override", Int64.Type}, 
        {"Points Calculated", Int64.Type}, 
        {"Area of preference", type text}, 
        {"Number of areas of preference", Int64.Type}, 
        {"Does this applicant require a significantly adapted property as recommended by the OT?", type text}, 
        {"Do you want or need any of the following types of accommodation? Housing with access for wheelchairs", type text}, 
        {"Do you want or need any of the following types of accommodation? Sheltered housing", type text}, 
        {"Do you want or need any of the following types of accommodation? Amenity Housing", type text}, 
        {"Type of property Maisonette (upper floor)", type text}, 
        {"Type of property Tenement (ground floor)", type text}, 
        {"Type of property Four in a block (upper floor)", type text}, 
        {"Type of property Maisonette (ground floor)", type text}, 
        {"Type of property Three-storey end-terrace", type text}, 
        {"Type of property Hostel", type text}, 
        {"Type of property Tenement (upper floor)", type text}, 
        {"Type of property Basement flat", type text}, 
        {"Type of property Three-storey mid-terrace", type text}, 
        {"Type of property Semi-detached house", type text}, 
        {"Type of property Mid-terrace house", type text}, 
        {"Type of property End-terrace house", type text}, 
        {"Type of property Multi-storey flat", type text}, 
        {"Type of property Four in a block (ground floor)", type text}, 
        {"Type of property Detached house", type text}, 
        {"Type of property Bungalow", type text}, 
        {"Size of property Three bedrooms", type text}, 
        {"Size of property Four bedrooms", type text}, 
        {"Size of property Five or more bedrooms", type text}, 
        {"Size of property Bedsit (no bedrooms)", type text}, 
        {"Size of property One bedroom", type text}, 
        {"Size of property Two bedrooms", type text},
        {"", type text}
    }),

    // Add weighted proportion column → key part for proportionate demand
    #"Added Proportion" = Table.AddColumn(#"Changed Type", "Proportion", each 
        if [Number of areas of preference] <> null 
            and [Number of areas of preference] > 0 
        then 1 / [Number of areas of preference] 
        else null,
        type number
    )

in
    #"Added Proportion"




    //table

    Sum of Proportion	Column Labels								
Row Labels	1	2	3	4	5	6	7	8	Grand Total
Ardrossan	135	38	33	20	8	2	1	0	238
Arran	110	19	17	6	1	1	1		155
Beith	43	10	19	5	2	0	0		79
Cumbrae	17	4	0	0	0		0		21
Dalry	61	12	13	6	2	1	0		95
Irvine	785	195	213	115	42	9	7	1	1366
Kilbirnie	80	21	18	11	5	4	1		140
Kilwinning	272	68	69	47	14	5	2		477
Largs	176	36	58	20	8	0	1		299
Outwith North Ayrshire	0								0
Saltcoats	164	45	34	23	13	2	1	0	283
Stevenston	122	32	23	16	7	1	1	1	202
Grand Total	1966	479	497	269	102	27	13	2	3355



//instuctions 
Step	Detail
1	Created copy of original data
2	Added proportion column to data, column AY = 1/Number of Areas of Preference (Column E)
3	Filter Column AM by Band 1 and give all Homeless applicants -5 points in points override field (Column B)
4	Add locality information filter column M and use values in table opposite to populate Locality column AZ
5	Created pivot table 'Proportionate Pivot'


in the proportionate pivot is 
filters: points override, bandID, property adaptations (i've implemented these as a slicer)
columns - strategic bedroom need
rows - locality 
values - sum of proportion 

Step	Detail
1	Created copy of original data
2	Removed duplicates using Housing Register Reference Number (column AU)
3	Filter Column AM by Band 1 and give all Homeless applicants -5 points in points override field (Column B)
4	Add current region (column AZ) - region applicant currently resides within
5	Filter data by post code (see list in columns D&E) and sense check locality information, errors / queries highlighted
6	Review postcodes not known and those issued in error etc, sense check heading filter lists
7	Add current locality (Column BA) - using information in table 2 opposite
8	Create pivot table 'AD Pivot'

ad pivot is 
filters: points override, bandID, property adaptations (i've implemented these as a slicer)
columns - strategic bedroom need
rows - current locality 
values - sum of proportion 


Table 1: Current region from applicants current Post Codes				
Locality	Post Codes			
Ardrossan	KA22			
Arran	KA27			
Beith	KA15			
Cumbrae	KA28			
Dalry	KA24			
Irvine	KA11 & KA12			
Kilbirnie	KA25 & KA14 Longbar & Glengarnock			
Kilwinning	KA13			
Largs	KA30 & KA29 (Fairlie) & PA17 (Skelmorlie) & KA23 (West Kilbride)			
Outwith North Ayrshire	All others			
Saltcoats	KA21			
Stevenston	KA20			
				
Table 2: Locality from applicants Current Region				
Locality	Region			
Arran	Arran			
Cumbrae	Cumbrae			
Garnock Valley	Beith			
	Dalry			
	Kilbirnie			
Irvine	Irvine			
Kilwinning	Kilwinning			
North Coast	Largs			
Three Towns	Ardrossan			
	Saltcoats			
	Stevenston			
Z - Outwith NA	Outwith North Ayrshire			
				
				
				
				
	KA22	Ardrossan		
	KA27	Arran		
	KA15	Beith		
	KA28	Cumbrae		
	KA24	Dalry		
	KA11	Irvine		
	KA12	Irvine		
	KA14	Kilbirnie		
	KA25	Kilbirnie		
	KA13	Kilwinning		
	KA29	Largs		
	KA23	Largs		
	PA17	Largs		
	KA21	Saltcoats		
	KA20	Stevenston		
				
				
Client Region	Post Code		Locality	Region
Ardrossan	KA22		ARRAN	ARRAN
Arran	KA27		CUMBRAE	CUMBRAE
Beith	KA15		GARNOCK VALLEY	BEITH
Cumbrae	KA28		GARNOCK VALLEY	DALRY
Dalry	KA24		GARNOCK VALLEY	KILBIRNIE
Irvine	KA11		IRVINE	IRVINE
Irvine	KA12		KILWINNING	KILWINNING
Kilbirnie	KA14		NORTH COAST	LARGS
Kilbirnie	KA25		THREE TOWNS	ARDROSSAN
Kilwinning	KA13		THREE TOWNS	SALTCOATS
Largs	KA29		THREE TOWNS	STEVENSTON
Largs	KA30		Z - OUTWITH NA	OUTWITH NORTH AYRSHIRE
Largs	KA23			
Largs	PA17			
Outwith North Ayrshire	All others			


pyhton script

import pandas as pd
import numpy as np
import re

# Power BI provides the incoming table as 'dataset' - copy it to df
df = dataset.copy()

# --- Safety checks ---------------------------------------------------------
required_cols = ['Area of preference', 'Number of areas of preference', 'Correspondence PostCode', 'HousingRegisterRef']
missing = [c for c in required_cols if c not in df.columns]
if missing:
    # If Number of areas is missing that's okay (we can compute it). Otherwise error message in outputs.
    pass

# --- Helper: clean postcode and build prefix --------------------------------
def clean_postcode(pc):
    if pd.isna(pc):
        return ''
    s = str(pc).upper()
    # keep alnum and spaces, then remove spaces
    s = re.sub(r'[^A-Z0-9]', '', s)
    # take first 4 characters as prefix (similar to your Power Query behaviour)
    return s[:4]

df['__PostcodePrefix'] = df['Correspondence PostCode'].apply(clean_postcode)

# --- Postcode prefix -> Locality & Region mapping (from your tables) -------
# Mapping built from the mapping tables you supplied. Add or edit as necessary.
postcode_to_locality_region = {
    # Table 1 / list mapping (prefix: (Locality, Region))
    'KA22': ('Ardrossan', 'Ardrossan'),
    'KA27': ('Arran', 'Arran'),
    'KA15': ('Beith', 'Beith'),
    'KA28': ('Cumbrae', 'Cumbrae'),
    'KA24': ('Dalry', 'Dalry'),
    'KA11': ('Irvine', 'Irvine'),
    'KA12': ('Irvine', 'Irvine'),
    'KA14': ('Kilbirnie', 'Kilbirnie'),
    'KA25': ('Kilbirnie', 'Kilbirnie'),
    'KA13': ('Kilwinning', 'Kilwinning'),
    'KA29': ('Largs', 'Largs'),
    'KA30': ('Largs', 'Largs'),
    'KA23': ('Largs', 'Largs'),
    'PA17': ('Largs', 'Largs'),
    'KA21': ('Saltcoats', 'Saltcoats'),
    'KA20': ('Stevenston', 'Stevenston'),
    # If you need more prefixes, add here. All others -> Outwith North Ayrshire
}

# Map prefixes
def map_prefix(pfx):
    if not pfx:
        return ('', '')
    if pfx in postcode_to_locality_region:
        return postcode_to_locality_region[pfx]
    # Some prefixes may be shorter/longer, try first 3 chars fallback
    short = pfx[:3]
    if short in postcode_to_locality_region:
        return postcode_to_locality_region[short]
    return ('Outwith North Ayrshire', 'Outwith North Ayrshire')

mapped = df['__PostcodePrefix'].apply(lambda x: map_prefix(x))
df[['MappedLocality', 'MappedRegion']] = pd.DataFrame(mapped.tolist(), index=df.index)

# --- Ensure Number of areas exists: compute if blank or zero ----------------
# If 'Number of areas of preference' is missing or has invalid values, compute from AOP grouping
if 'Number of areas of preference' not in df.columns or df['Number of areas of preference'].isnull().any():
    # Count non-empty distinct or non-null AOPs per applicant
    # Use the raw Area of preference values, trim them
    df['__AOP_clean'] = df['Area of preference'].astype(str).str.strip()
    # treat empty strings and literal 'nan' as missing
    df.loc[df['__AOP_clean'].isin(['', 'nan', 'None', 'NoneType']), '__AOP_clean'] = np.nan
    numaops = df.groupby('HousingRegisterRef')['__AOP_clean'].nunique(dropna=True).rename('Computed_NumAOPs')
    df = df.merge(numaops, left_on='HousingRegisterRef', right_index=True, how='left')
    # where original col exists and is valid use it, otherwise use computed
    if 'Number of areas of preference' in df.columns:
        df['Number of areas of preference'] = df['Number of areas of preference'].where(
            pd.notnull(df['Number of areas of preference']) & (df['Number of areas of preference'] > 0),
            df['Computed_NumAOPs']
        ).fillna(0).astype(int)
    else:
        df['Number of areas of preference'] = df['Computed_NumAOPs'].fillna(0).astype(int)
    # drop helper
    df.drop(columns=['__AOP_clean', 'Computed_NumAOPs'], inplace=True, errors='ignore')
else:
    # ensure integer type
    df['Number of areas of preference'] = pd.to_numeric(df[']()_



