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





    //AOP proportion

    #"Added Proportion" = Table.AddColumn(#"Previous Step", "Proportion", each 
    if [Number of areas of preference] <> null and [Number of areas of preference] > 0 
    then 1 / [Number of areas of preference] 
    else null
)

in
    #"Added Proportion"
