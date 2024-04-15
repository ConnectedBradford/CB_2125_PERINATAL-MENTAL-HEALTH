# Output Code

# Output Code

###perinatal maternal health cohort
## Date: 28/11/2023
##Author: Enass Duro
## two data set will be formed one for the ctv3 codes and the other for the mendication
## key date will be date_of_pregnancy_outcome_current_fetus
##CTV3 codes dataset
create or replace table `yhcr-prd-phm-bia-core.CB_MYSPACE_ED.CVT3`
as
SELECT DISTINCT
person_id as person_id
, dateevent as dateevent
, ctv3code as ctv3code
, ctv3text as ctv3text
FROM `yhcr-prd-phm-bia-core.CB_FDM_PrimaryCare_V8.tbl_srcode`;

create or replace table `yhcr-prd-phm-bia-core.CB_MYSPACE_ED.medication`
as
SELECT DISTINCT
person_id as person_id
, nameofmedication as nameofmedication
, datemedicationstart as datemedicationstart
, datemedicationend as datemedicationend
FROM `yhcr-prd-phm-bia-core.CB_FDM_PrimaryCare_V8.tbl_srprimarycaremedication`;

##combined both so we can know which medication were given at each visit
create or replace table `yhcr-prd-phm-bia-core.CB_MYSPACE_ED.both`
as select 
a.person_id as person_id
,a.dateevent as dateevent
, a.ctv3code as ctv3code
, a.ctv3text as ctv3text
, b.nameofmedication as nameofmedication
, b.datemedicationstart as datemedicationstart
, b.datemedicationend as datemedicationend 
FROM `yhcr-prd-phm-bia-core.CB_MYSPACE_ED.CVT3`a
left join `yhcr-prd-phm-bia-core.CB_MYSPACE_ED.medication`b
on a.person_id= b.person_id AND a.src_dateevent= b.src_datemedicationstart; 

################################# create maternal cohort
create or replace table `yhcr-prd-phm-bia-core.CB_MYSPACE_ED.perinatal_mental`
as
SELECT 
person_id as person_id

, last_menstrual_period_date as last_menstrual_period_date
, pregnancy_first_contact_date as pregnancy_first_contact_date
,discharge_date_mother_maternity_services as discharge_date_mother_maternity_services
,support_status_mother_at_booking as support_status_mother_at_booking
,physical_disability_status_indicator_mother_at_booking as physical_disability_status_indicator_mother_at_booking
, physical_disability_code_mother_at_booking as physical_disability_code_mother_at_booking
, mental_health_prediction_and_detection_indicator_mother_at_booking as   mental_health_prediction_and_detection_indicator_mother_at_booking 
, complex_social_factors_indicator_mother_at_booking as complex_social_factors_indicator_mother_at_booking
, person_height_in_meters_mother_at_booking as person_height_in_meters_mother_at_booking
,person_weight_in_kilograms_mother_at_booking as person_weight_in_kilograms_mother_at_booking
, substance_use_status_mother_at_booking as substance_use_status_mother_at_booking, 
employment_status_mother_at_booking as employment_status_mother_at_booking,
employment_status_mother_at_booking_desc as employment_status_mother_at_booking_desc
, smoking_status_mother_at_booking as smoking_status_mother_at_booking
,smoking_status_mother_at_booking_desc as smoking_status_mother_at_booking_desc
, status_of_folic_acid_supplement_mother_at_booking as status_of_folic_acid_supplement_mother_at_booking
, weekly_alcohol_units_mother_at_booking as weekly_alcohol_units_mother_at_booking
 
FROM `yhcr-prd-phm-bia-core.CB_FDM_Maternity.maternity_delivery`
WHERE person_id is not null;


#### to use the date of pregnancy out come as key date we need to joint the maternity_birth with the delivery file but the person Id in the maternity birth is the child person id not the mother so we need to change that 
 create or replace table `yhcr-prd-phm-bia-core.CB_MYSPACE_ED.perinatal_mental2`
as 

SELECT 
a.person_id as person_id
,a.baby_year_of_birth as baby_year_of_birth
,a.baby_month_of_birth as baby_month_of_birth
,a.date_of_pregnancy_outcome_current_fetus as date_of_pregnancy_outcome_current_fetus
, a.fetal_order_fetus_outcome as  fetal_order_fetus_outcome
, a.pregnancy_outcome_current_fetus_desc as pregnancy_outcome_current_fetus_desc
,SAFE_CAST(b.mother_person_id AS INT64) as mother_person_id 

FROM `yhcr-prd-phm-bia-core.CB_FDM_Maternity.maternity_birth`a
LEFT  JOIN  
`yhcr-prd-phm-bia-core.CB_FDM_Maternity.mother_child_relationship` b
 
ON SAFE_CAST(a.person_id AS INT64)  = SAFE_CAST(b.child_person_id AS INT64);

##change person_id to the maternal id 
create or replace table `yhcr-prd-phm-bia-core.CB_MYSPACE_ED.perinatal_mental12`
AS
SELECT 
mother_person_id as person_id
,person_id as child_person_id
,baby_year_of_birth as baby_year_of_birth
,baby_month_of_birth as baby_month_of_birth
,date_of_pregnancy_outcome_current_fetus as date_of_pregnancy_outcome_current_fetus
, fetal_order_fetus_outcome as  fetal_order_fetus_outcome
, pregnancy_outcome_current_fetus_desc as pregnancy_outcome_current_fetus_desc

FROM `yhcr-prd-phm-bia-core.CB_MYSPACE_ED.perinatal_mental2`;

##combined the two files

create or replace table `yhcr-prd-phm-bia-core.CB_MYSPACE_ED.mother_baby`
as 
SELECT 
a.person_id as person_id
, a.last_menstrual_period_date as last_menstrual_period_date
, a.pregnancy_first_contact_date as pregnancy_first_contact_date
,a.discharge_date_mother_maternity_services as discharge_date_mother_maternity_services
,a.support_status_mother_at_booking as support_status_mother_at_booking
,a.physical_disability_status_indicator_mother_at_booking as physical_disability_status_indicator_mother_at_booking
, a.physical_disability_code_mother_at_booking as physical_disability_code_mother_at_booking
, a.mental_health_prediction_and_detection_indicator_mother_at_booking as   mental_health_prediction_and_detection_indicator_mother_at_booking 
, a.complex_social_factors_indicator_mother_at_booking as complex_social_factors_indicator_mother_at_booking
, a.person_height_in_meters_mother_at_booking as person_height_in_meters_mother_at_booking
,a.person_weight_in_kilograms_mother_at_booking as person_weight_in_kilograms_mother_at_booking
, a.substance_use_status_mother_at_booking as substance_use_status_mother_at_booking, 
a.employment_status_mother_at_booking as employment_status_mother_at_booking,
a.employment_status_mother_at_booking_desc as employment_status_mother_at_booking_desc
, a.smoking_status_mother_at_booking as smoking_status_mother_at_booking
,a.smoking_status_mother_at_booking_desc as smoking_status_mother_at_booking_desc
, a.status_of_folic_acid_supplement_mother_at_booking as status_of_folic_acid_supplement_mother_at_booking
, a.weekly_alcohol_units_mother_at_booking as weekly_alcohol_units_mother_at_booking
,b.child_person_id as child_person_id
, b.baby_year_of_birth as baby_year_of_birth
, b.baby_month_of_birth as baby_month_of_birth
,b.date_of_pregnancy_outcome_current_fetus as date_of_pregnancy_outcome_current_fetus
, b.fetal_order_fetus_outcome as  fetal_order_fetus_outcome
, b.pregnancy_outcome_current_fetus_desc as pregnancy_outcome_current_fetus_desc
FROM `yhcr-prd-phm-bia-core.CB_MYSPACE_ED.perinatal_mental`a
LEFT  JOIN  
`yhcr-prd-phm-bia-core.CB_MYSPACE_ED.perinatal_mental12` b

ON 
a.person_id = b.person_id
WHERE a.person_id IS NOT NULL ;



###############################################



create or replace table `yhcr-prd-phm-bia-core.CB_MYSPACE_ED.mother_baby_race`
as 
SELECT 

 a.person_id as person_id
, a.last_menstrual_period_date as last_menstrual_period_date
, a.pregnancy_first_contact_date as pregnancy_first_contact_date
,a.discharge_date_mother_maternity_services as discharge_date_mother_maternity_services
,a.support_status_mother_at_booking as support_status_mother_at_booking
,a.physical_disability_status_indicator_mother_at_booking as physical_disability_status_indicator_mother_at_booking
, a.physical_disability_code_mother_at_booking as physical_disability_code_mother_at_booking
, a.mental_health_prediction_and_detection_indicator_mother_at_booking as   mental_health_prediction_and_detection_indicator_mother_at_booking 
, a.complex_social_factors_indicator_mother_at_booking as complex_social_factors_indicator_mother_at_booking
, a.person_height_in_meters_mother_at_booking as person_height_in_meters_mother_at_booking
,a.person_weight_in_kilograms_mother_at_booking as person_weight_in_kilograms_mother_at_booking
, a.substance_use_status_mother_at_booking as substance_use_status_mother_at_booking, 
a.employment_status_mother_at_booking as employment_status_mother_at_booking,
a.employment_status_mother_at_booking_desc as employment_status_mother_at_booking_desc
, a.smoking_status_mother_at_booking as smoking_status_mother_at_booking
,a.smoking_status_mother_at_booking_desc as smoking_status_mother_at_booking_desc
, a.status_of_folic_acid_supplement_mother_at_booking as status_of_folic_acid_supplement_mother_at_booking
, a.weekly_alcohol_units_mother_at_booking as weekly_alcohol_units_mother_at_booking
, a.child_person_id as child_person_id
, a.baby_year_of_birth as baby_year_of_birth
, a.baby_month_of_birth as baby_month_of_birth
,a.date_of_pregnancy_outcome_current_fetus as date_of_pregnancy_outcome_current_fetus
, a.fetal_order_fetus_outcome as  fetal_order_fetus_outcome
, a.pregnancy_outcome_current_fetus_desc as pregnancy_outcome_current_fetus_desc

,b.birth_datetime as birth_date
, b.ethnicity_source_value as ethnicity_source_value
, b.race_source_value as race_source_value
FROM `yhcr-prd-phm-bia-core.CB_MYSPACE_ED.mother_baby`a
LEFT  JOIN  
`yhcr-prd-phm-bia-core.CB_FDM_Maternity.person` b
ON 
a.person_id = b.person_id
WHERE a.person_id IS NOT NULL AND a.date_of_pregnancy_outcome_current_fetus is NOT NULL ;




### combined with primary care data

create or replace table `yhcr-prd-phm-bia-core.CB_MYSPACE_ED.pregnancy_pri`
as 
SELECT 

 a.person_id as person_id
,a.child_person_id as child_person_id

,CAST(a.birth_date AS DATE) as mother_birth_date
,b.dateevent as dateevent
, b.ctv3code as ctv3code
, b.ctv3text as ctv3text
, b.nameofmedication as nameofmedication
FROM `yhcr-prd-phm-bia-core.CB_MYSPACE_ED.mother_baby_race`a
LEFT   JOIN  
`yhcr-prd-phm-bia-core.CB_MYSPACE_ED.both` b
ON 
a.person_id = b.person_id
WHERE a.person_id IS NOT NULL; 


###################create event dates
create or replace table  `yhcr-prd-phm-bia-core.CB_MYSPACE_ED.pregnancy1_pri`
AS 
SELECT *
, date_add(date_of_pregnancy_outcome_current_fetus, INTERVAL 1 YEAR) AS postnatal_date1 
,date_sub(date_of_pregnancy_outcome_current_fetus, INTERVAL 9 MONTH) AS antenatal_date
, CAST(dateevent AS DATE)  as event_date
 ,  DATE_DIFF(date_of_pregnancy_outcome_current_fetus, mother_birth_date  , YEAR) AS mother_age
 , EXTRACT(YEAR FROM dateevent ) AS event_year


FROM `yhcr-prd-phm-bia-core.CB_MYSPACE_ED.pregnancy_pri`;


##select only CTV3 codes in the perinatal period



create or replace table  `yhcr-prd-phm-bia-core.CB_MYSPACE_ED.pregnancy1_pri`
AS 
SELECT *

FROM `yhcr-prd-phm-bia-core.CB_MYSPACE_ED.pregnancy1_pri`
WHERE event_date BETWEEN  antenatal_date  AND  postnatal_date1;






create or replace table  `yhcr-prd-phm-bia-core.CB_MYSPACE_ED.pregnancy1_pri`
AS 
SELECT *
, case when event_date < date_of_pregnancy_outcome_current_fetus then "antenatal" 
 WHEN event_date > date_of_pregnancy_outcome_current_fetus THEN "postnatal" END as event_period
 

FROM `yhcr-prd-phm-bia-core.CB_MYSPACE_ED.pregnancy1_pri`;



#######################################create pmh vararibles


create or replace table  `yhcr-prd-phm-bia-core.CB_MYSPACE_ED.pregnancy_final1`
AS 
SELECT *
, CASE WHEN ctv3code IN ('1685', '6651', '6652', '6653', '6654', '6658','6659', '1B13.' ,'1B17.', '225..', '62T1.', '665..', '69F..', '8A2..',
'8A21.', '8G10.', '8G21.', '8G9..', '8G91.', '8G9Z.', '8H23.', '8H230', '8H38.', '8H49.', '8H7A.', '8H7B.', '8H7Z.',
'8HB8.', '8HJ3.', '8HVO.', '9HZ..', '9N0T.', '9N1M.', '9N2B.', '9NJ1.', 'E....', 'E103.', 'E112.', 'E1121', 'E1122',
'E1123', 'E1124', 'E1125', 'E1126', 'E1131', 'E1132', 'E1134', 'E1136', 'E1137', 'E11y2', 'E11z.', 'E11z1', 'E12..',
'E120.', 'E12z.', 'E130.', 'E131.', 'E1y..', 'E20..', 'E200.', 'E2000', 'E2001', 'E2002', 'E2004', 'E2005', 'E200z',
'E201.', 'E2010', 'E2015', 'E201C', 'E202.', 'E2020', 'E2021', 'E2022', 'E2026', 'E2027', 'E2028', 'E2029', 'E202C',
'E202E', 'E203.', 'E2031', 'E203z', 'E204.', 'E205.', 'E206.', 'E207.', 'E20y0', 'E21..', 'E2111', 'E2112', 'E2113',
'E211z', 'E212.', 'E213.', 'E214z', 'E2152', 'E21y.', 'E21y1', 'E21y2', 'E27zz', 'E28..', 'E280.', 'E2830', 'E2831',
'E283z', 'E284.', 'E290.', 'E290z', 'E2920', 'E2924', 'E2B..', 'E2B1.', 'E2C0.', 'Eu...', 'Eu12y', 'Eu14.', 'Eu141',
'Eu142', 'Eu181', 'Eu19.', 'Eu20.', 'Eu231', 'Eu25.', 'Eu2y.', 'Eu2z.', 'Eu310', 'Eu312', 'Eu313', 'Eu314', 'Eu316',
'Eu317', 'Eu31z', 'Eu320', 'Eu321', 'Eu322', 'Eu32z', 'Eu33.', 'Eu330', 'Eu331', 'Eu33y', 'Eu4..', 'Eu40.', 'Eu400',
'Eu40z', 'Eu41.', 'Eu410', 'Eu412', 'Eu413', 'Eu41z', 'Eu420', 'Eu422', 'Eu42y', 'Eu42z', 'Eu430', 'Eu43y', 'Eu43z',
'Eu44.', 'Ez...', 'Ua16B', 'Ua18F', 'Ua18K', 'Ub0qs', 'X00Sb', 'X00Sc', 'X00Sf', 'X00SL', 'X00SO', 'X00SQ', 'X00SR',
'X00SS', 'X00SU', 'X40Dl','X40Dm', 'X40Dp', 'X71bp', 'X71Ec', 'X75ws', 'X760o', 'X760u', 'X7617', 'X761J', 'X761L',
'X761N', 'Xa02E', 'Xa0wV', 'Xa0XG', 'Xa0XJ', 'Xa0XO', 'Xa0XP', 'Xa0XQ', 'Xa0XX', 'Xa17z', 'Xa19B', 'Xa1e9', 'Xa1eL',
'Xa3aF', 'Xa3Xk', 'Xa4HV', 'Xa7kB', 'Xa7O5', 'Xa8Ik', 'Xaa8p', 'Xaa8q', 'Xaacs', 'XaAdM', 'XaAel', 'XaAem', 'XaAen',
'XaAfJ', 'XaAh4', 'XaAiE', 'XaAiI', 'XaAkB', 'XaAkI', 'XaAnb', 'XaAOd', 'XaAOe', 'XaAOh', 'XaAQo', 'XaAS4', 'XaAU5',
'XaAUA', 'XaaWr', 'XaAXe', 'XaAyL', 'XaAZI', 'Xab9E', 'Xab9F', 'XaB9J', 'XaBfp', 'XaBHK', 'XaBT1', 'XaBTD', 'Xabu5',
'XaBvV', 'XaBvX', 'XaC6O', 'XaCAw', 'XaCFD', 'XacHF', 'XacHZ', 'XaCIs', 'XaCIt', 'XaCIu', 'XaECG', 'XaEFB', 'XaF9U',
'XafjR', 'XagMs', 'XagMt', 'XagOI', 'XagSV', 'XaI8j', 'XaIkd', 'XaIkg', 'XaIKH', 'XaIku', 'XaImU', 'XaINQ', 'XaINy',
'XaIOg', 'XaIOh', 'XaIOi', 'XaIOl', 'XaIOq', 'XaIpA', 'XaIpB', 'XaIPw', 'XaITG', 'XaItx', 'XaIuR', 'XaIux', 'XaIvk',
'XaIvp', 'XaIvq', 'XaIwe', 'XaIwf', 'XaIWQ', 'XaIXg', 'XaIXh', 'XaIXi', 'XaIXs', 'XaIYN', 'XaIyU', 'XaIyv', 'XaJ4V',
'XaJ4w', 'XaJ4x', 'XaJOA', 'XaJON', 'XaJPu', 'XaJPv', 'XaJPz', 'XaJQ1', 'XaJQE', 'XaJQI', 'XaJQJ', 'XaJQS', 'XaJQT',
'XaJQW', 'XaJQY', 'XaJQZ', 'XaJr3', 'XaJRr', 'XaJuK', 'XaJuT', 'XaJuV', 'XaJuW', 'XaJWg', 'XaJWh', 'XaK1f', 'XaK5q',
'XaK5r', 'XaK6d', 'XaK6e', 'XaK6f', 'XaK6K', 'XaK70', 'XaKAx', 'XaKaX', 'XaKaY', 'XaKbb', 'XaKEz', 'XaKUk', 'XaKUM',
'XaL0o', 'XaL0q', 'XaL0r', 'XaL0s', 'XaL0t', 'XaL0u', 'XaL0w', 'XaL19', 'XaLBl', 'XaLCQ', 'XaLDN', 'XaLFe', 'XaLFk',
'XaLFL', 'XaLIb', 'XaLIc', 'XaLLG', 'XaLNF', 'XaLnp', 'XaLnq', 'XaLnr', 'XaLQw', 'XaM7s', 'XaMGL', 'XaMGz', 'XaMhM',
'XaMi4', 'XaMJ8', 'XaN3a', 'XaN4c', 'XaN4d', 'XaN4e', 'XaN4f', 'XaNkT', 'XaNlN', 'XaNPL', 'XaNTc', 'XaObo', 'XaONq',
'XaOxM', 'XaP7x', 'XaP8d', 'XaPTT', 'XaPvw', 'XaPvy', 'XaQBz', 'XaQC0', 'XaQvz', 'XaQWJ', 'XaR9y', 'XaX04','XaX0C',
'XaXe3', 'XaXEJ', 'XaXH8', 'XaXHm', 'XaXiE', 'XaXiH', 'XaXl2', 'XaXwe', 'XaY2C', 'XaY4X', 'XaYgS', 'XaYwr', 'XaZ2p',
'XaZcf', 'XaZIW', 'XaZJQ', 'XE0re', 'XE1aW', 'XE1Y0', 'XE1YH', 'XE1Zb', 'XE1Zf', 'XM011', 'XM0Ar', 'XM0d1', 'Y05b8',
'Y05b9', 'Y05bc', 'Y120f', 'Y1b97'
) THEN 'yes' ELSE 'no' END AS PMH
, 
CASE WHEN ctv3code IN ('Eu412','X00Sb','XE1aW','Eu41.','Eu413','Eu40z','Eu40.','Xa0XQ','Xa0XX','Xa0XJ','Xa0XG','Xa0XO','Xa0XP',
'X761N','Xa7kB','XaECG','E200.','X00Sc','Ub0qs','E200z','E2000','1B13.','E2004','E2002','Xa3Xk',
'Eu41z','XaP8d','Eu314','1B17.','E204.','Eu32z','Eu313','XE1Zb','Eu320','Eu321','Eu33y','Eu33.',
'Eu330','Eu331','XE1Zf','Eu322','X00SQ','XaY2C','XaZ2p','XaXe3','XaR9y','E11y2','E290.','E290z','E2B1.',
'E206.','XE0re','XaK6d','XaK6f','XaItx','XaK6e','XaMGL','XaB9J','X761J','E112.','X00SO','E2B..','X760u',
'X00SR','X00SS','XaLFe','X00SU','XaCIs','X40Dl','XaCIt','XaJWh','62T1.','E130.','Xa0wV','E1137',
'E1136','E1134','E1131','E1132','XaL0r','XaYgS','XaCIu','X40Dm','XE1Y0','E1126','E1121','E1125','E1124',
'E1122','E1123','XaX0C','XaImU','XaObo','Eu42y','E2C..','E213.','E205.','E2001','Eu2z.','Eu2y.',
'Eu430','Eu231','Eu400','Eu312','Eu317','Eu31z','Eu44.','Eu19.','Eu12y','Eu142','Eu141','Eu181','Eu...',
'Eu14.','Eu422','Eu4..','Eu42z','Eu43y','Eu410','Eu420','Eu43z','E2026','E131.','XaJPu','E280.','E2831',
'E28..','E2830','E2924','XaK5q','XaJPv','8H23.','XaIyv','E211z','E2C0.','E2021','E2022','Xaa8p','XaJQ1',
'XE1YH','E2027','E21y1','Eu316','Eu310','E21y2','XaBvX','XaEFB','E2028','XaIvq','E214z','X71Ec','XaIku',
'Xa8Ik','XaIOi','1685','XM0Ar','XaAyL','E2113','E202C','Xa3aF','XaLCQ','XaK1f','XaIuR','XaXHm','XaAkI',
'XaAiE','XaAiI','XaAkB','XaAnb','XaOxM','XaL0w','XaNTc','XaXH8','XaLQw','XacHZ','E2015','E201.','XaLIb',
'XaKUk','E2112','XaIOg','XaITG','8H230','XaNlN','Ua18K','E2152','XaJ4x','XaJ4w','XaJ4V','8G21.','XacHF',
'E2029','E202E','Xa02E','6654','XaQC0','XaF9U','XaINQ','XaJQT','E207.','X00SL','E2111','E2010',
'9NJ1.','XaLnq','XaLnp','XaLnr','6653','XaQBz','Y05b8','Y05b9','XaKUM','Ez...','9HZ..','XaJON','XaXiE',
'XaBHK','XaIXs','XaJQE','Xa4HV','XaPTT','XaK5r','XaJPz','XaC6O','XaCAw','XaIvp','XaJQI','XaIOl','XaJQJ',
'E....','XaJQS','XaLBl','XaX04','XaJr3','XaJuT','XaJuK','XaJuV','XagOI','XaMGz','XaJuW','XaKbb','XaK70',
'XagSV','XaJWg','XaIyU','XaMJ8','XaBfp','8HB8.','E20..','XaIXi','XaIXh','8H38.','E2031','E203.','E203z',
'XaINy','XaIWQ','E283z','E11z.','E21y.','8G9..','8G9Z.','E12..','E12z.','E103.','XaKEz','XaXEJ','Xa19B',
'XaIOq','XaIXg','E21..','E2020','E202.','XaIOh','XaIpA','XaaWr','X00Sf','8HVO.','XaZIW','E201C','665..',
'8A2..','8A21.','8HJ3.','6651','6658','6659','E27zz','X71bp','8G10.','E11z1','E2005','XaPvy','8H7A.',
'8H7Z.','XaJQZ','XaAZI','XaNPL','XaL0q','XaLNF','XaAen','XaL0s','Y05bc','XaIkd','XaXl2','8H7B.','XaAel',
'XaBT1','XaZcf','XaPvw','XaAem','XaAfJ','XaIkg','XaJQY','XaIPw','XaL0o','XaLFL','XaLFk','XaMhM','XaAh4',
'8H49.','XaBTD','XaBvV','XaL19','XaAdM','XaQWJ','Xaa8q','Eu25.','E212.','Eu20.','X761L','Xa17z','XaN3a',
'XaAUA','9N2B.','XaP7x','XaAS4','XaL0u','XaL0t','XaM7s','XaAU5','XaCFD','XaAXe','9N0T.','XaONq','9N1M.',
'E2920','X40Dp','XagMt','XagMs','E120.','XaQvz','E20y0','XaK6K','XaN4c','XaN4d','XaN4e','XaN4f','XaI8j',
'E284.','Ua18F','XaKAx','8G91.','XaIux','XaJQW','XaAOe','XaAQo','XaAOd','XaAOh','XaJRr','XaIvk','XaJOA',
'X7617','Xa1eL','E1y..') THEN 'yes' ELSE 'no' END AS PMH_problems
,  
CASE WHEN ctv3code IN ('XaL0r','XaYgS','XaIpB','XaJWh','Eu412','X00Sb','1B17.','E204.','Eu32z','XE1Zb','Eu320','Eu321','Eu33y','Eu33.','Eu330','Eu331','XE1Zf',
'Eu322','X00SQ','XaY2C','XaZ2p','XaXe3','XaR9y','E11y2','E290.','E290z','E2B1.','E206.','XE0re','XaK6d','XaK6f','XaItx','XaK6e','XaMGL',
'XaB9J','X761J','E112.','X00SO','E2B..','X760u','X00SR','X00SS','XaLFe','X00SU','XaCIs','X40Dl','XaCIt','62T1.','E130.','Xa0wV','E1137',
'E1136','E1134','E1131','E1132','XaCIu','X40Dm','XE1Y0','E1126','E1121','E1125','E1124','E1122','E1123','XaX0C','XaImU','E11z1','E205.',
'Eu430','XaKUk','E2112','X761L','Xa17z')
THEN 'yes' ELSE 'no' END AS Depression
, 
CASE WHEN ctv3code IN('Eu412','X00Sb','Eu42y','E2001','Eu400','Eu422','Eu42z','Eu410','Eu420','E2026','E131.','E280.','E2924','E2021',
'E2022','XE1YH','E2028','E202C','E2029','E202E','E207.','E20..','E2031','E203.','E203z','Xa19B','E2020','E202.',
'E2005','XE1aW','Eu41.','Eu413','Eu40z','Eu40.','Xa0XQ','Xa0XX','Xa0XJ','Xa0XG','Xa0XO','Xa0XP','X761N','Xa7kB',
'XaECG','E200.','X00Sc','Ub0qs','E200z','E2000','1B13.','E2004','E2002','Xa3Xk','Eu41z','XaP8d','E2920') THEN 'yes' ELSE 'no' END AS Anxiety,

CASE WHEN ctv3code IN ('6651','6658','XaBfp','XaIvq','XaKAx','Eu314','Eu313','XagSV','E2C..','E213.','Eu2z.','Eu2y.','Eu231',
'Eu312','Eu317','Eu31z','Eu44.','Eu19.','Eu12y','Eu142','Eu141','Eu181','Eu...','Eu14.','Eu4..','Eu43y',
'Eu43z','E2831','E28..','E2830','E211z','E2C0.','E2027','E21y1','Eu316','Eu310','E21y2','XaEFB','E214z',
'1685','XM0Ar','XaAyL','E2113','E2015','E201.','Ua18K','E2152','Xa02E','XaF9U','X00SL','E2111','E2010',
'Ez...','9HZ..','E....','XaIWQ','E283z','E11z.','E21y.','E12..','E12z.','E103.','E21..','X00Sf','E201C',
'E27zz','Eu25.','E212.','Eu20.','X40Dp','E120.','E20y0','E284.','Ua18F','XaIux','X7617','Xa1eL','E1y..')
THEN 'yes' ELSE 'no' END AS PMH_others
,
CASE WHEN ctv3code IN ('XaL0r','XaYgS','XaIpB','9NJ1.','XaLnq','XaIXi','8A21.','8HJ3.','XaIyv','Xaa8p','X71Ec','XaIku',
'Xa8Ik','XaLQw','XacHZ','8G21.','XacHF','XaQC0','XaJQT','XaLnp','XaLnr','XaQBz','Y05b8','Y05b9',
'XaC6O','XaCAw','XaLBl','XaX04','XaJr3','XaJuT','XaMGz','XaKbb','XaK70','XaIyU','XaMJ8','8HB8.',
'XaKEz','XaIOh','XaIpA','XaaWr','8HVO.','XaZIW','6659','X71bp','8G10.','8H7A.','XaL0q','XaLNF',
'XaAen','XaL0s','XaAel','XaBT1','XaZcf','XaPvw','XaAem','XaAfJ','8G9..','8G9Z.','XaL0o','XaLFL',
'XaLFk','XaMhM','XaAh4','8H49.','XaBTD','XaBvV','XaL19','XaAdM','XaQWJ','XaN3a','XaAUA','9N2B.',
'XaP7x','XaAS4','XaL0u','XaL0t','XaM7s','XagMt','XagMs','XaK6K','XaN4c','XaN4d','XaN4e','XaI8j',
'8G91.','XaJQW','XaAOe','XaAQo','XaAOd','XaAOh','XaJRr','XaIvk','XaJOA') THEN 'Care mild'
 WHEN ctv3code IN ('XaIvq','XaKAx','XaIXh','XaJPu','XaK5q','XaJPv','8H23.','XaJQ1','XaIOi','Xa3aF',
'XaIOg','XaITG','8H230','XaNlN','XaINQ','6653','XaIXs','Xa4HV','XaPTT','XaIvp',
'XaJQI','XaIOl','XaJQJ','XaJQS','8H38.','XaINy','XaXEJ','XaIOq','XaIXg','665..',
'8A2..','XaJQZ','XaIkd','XaXl2','8H7B.','XaIkg','XaJQY','XaIPw','69F..','XaN4f') THEN 'Care sever' ELSE "no" END AS PMH_care
,

CASE WHEN ctv3code IN ('XaL0r','XaYgS','XaIpB','9NJ1.','XaLnq','XaIXi','8A21.','8HJ3.',
'XaIyv','Xaa8p','X71Ec','XaIku','Xa8Ik','XaLQw','XacHZ','8G21.',
'XacHF','XaQC0','XaJQT','XaLnp','XaLnr','XaQBz','Y05b8','Y05b9',
'XaC6O','XaCAw','XaLBl','XaX04','XaJr3','XaJuT','XaMGz','XaKbb','XaK70',
'XaIyU','XaMJ8','8HB8.','XaKEz','XaIOh','XaIpA','XaaWr','8HVO.','XaZIW',
'6659','X71bp','8G10.','8H7A.','XaL0q','XaLNF','XaAen','XaL0s','XaAel',
'XaBT1','XaZcf','XaPvw','XaAem','XaAfJ','8G9..','8G9Z.','XaL0o','XaLFL',
'XaLFk','XaMhM','XaAh4','8H49.','XaBTD','XaBvV','XaL19','XaAdM','XaQWJ',
'XaN3a','XaAUA','9N2B.','XaP7x','XaAS4','XaL0u','XaL0t','XaM7s','XagMt',
'XagMs','XaK6K','XaN4c','XaN4d','XaN4e','XaI8j','8G91.','XaJQW','XaAOe',
'XaAQo','XaAOd','XaAOh','XaJRr','XaIvk','XaJOA','XaIvq','XaKAx','XaIXh',
'XaJPu','XaK5q','XaJPv','8H23.','XaJQ1','XaIOi','Xa3aF','XaIOg','XaITG',
'8H230','XaNlN','XaINQ','6653','XaIXs','Xa4HV','XaPTT','XaIvp','XaJQI',
'XaIOl','XaJQJ','XaJQS','8H38.','XaINy','XaXEJ','XaIOq','XaIXg','665..',
'8A2..','XaJQZ','XaIkd','XaXl2','8H7B.','XaIkg','XaJQY','XaIPw','69F..',
'XaN4f','XaBvX','XaLCQ','XaK1f','XaIuR','XaXHm','XaAkI','XaAiE','XaAiI',
'XaAkB','XaAnb','XaOxM','XaL0w','XaNTc','XaXH8','XaLIb','XaJ4x','XaJ4w',
'XaJ4V','6654','XaJON','XaXiE','XaBHK','XaJQE','XaK5r','XaJPz','XaJuK',
'XaJuV','XagOI','XaJuW','XaJWg','XaPvy','8H7Z.','XaAZI','XaNPL','Y05bc',
'XaObo','Xaa8q','XaAU5','XaCFD','XaAXe','9N0T.','XaONq','9N1M.','XaQvz')
THEN 'yes' ELSE 'no' END AS PMH_mangment
,
 CASE WHEN ctv3code IN ('XM0d1','XaIwe','XaIwf','XaKaZ','Xab9E','XaZJQ','XaLIc','XafjR','Xaacs','Xab9F',
'XaLDN','XaY4X','XaLLG','XaKaX','XaKaY','Ua16B','XaYwr','X75ws','XaXwe','Xa1e9', 'XaNkT','Y1b97',
'69F..','XaIYN','XaXiH','Y120f','6652','XM011','X760o','225..','Xa7O5', 'XM0d1', 'XaMi4', 'XaIKH','Xabu5')
THEN 'yes' ELSE 'no' END AS PMH_screening
,

CASE WHEN ctv3code IN ('128..', 'ZV170', 'ZV1A.', 'ZV1A2', 'XaN3u')THEN 'yes' ELSE 'no' END AS Family_history 
, CASE WHEN nameofmedication IN ('Buspirone 5mg tablets','Circadin 2mg modified-release tablets (Flynn Pharma Ltd)','Citalopram 10mg tablets','Citalopram 20mg tablets','Citalopram 20mg tablets (A A H Pharmaceuticals Ltd)',
'Citalopram 20mg tablets (Teva UK Ltd)','Citalopram 40mg tablets','Citalopram 40mg tablets (Zentiva Pharma UK Ltd)','Citalopram 40mg/ml oral drops sugar free','Clomipramine 50mg capsules',
'Clonazepam 500microgram tablets','Depakote 250mg gastro-resistant tablets (Sanofi)','diazepam - Oral','Diazepam 10mg tablets','Diazepam 2mg tablets','Diazepam 5mg tablets','Dosulepin 25mg capsules',
'Escitalopram 10mg tablets','Escitalopram 20mg tablets','Escitalopram 20mg tablets (Actavis UK Ltd)','Escitalopram 5mg tablets','Fluoxetine 10mg capsules','Fluoxetine 10mg tablets',
'Fluoxetine 20mg capsules','Fluoxetine 20mg capsules (A A H Pharmaceuticals Ltd)','Fluoxetine 20mg capsules (Almus Pharmaceuticals Ltd)','Fluoxetine 20mg dispersible tablets sugar free',
'Fluoxetine 20mg/5ml oral solution','Fluoxetine 20mg/5ml oral solution sugar free','Fluoxetine 30mg capsules','Fluoxetine 40mg capsules','Fluoxetine 60mg capsules','Fluvoxamine 100mg tablets',
'Fluvoxamine 50mg tablets','Imipramine 25mg tablets','Lorazepam 1mg tablets','Lorazepam 500microgram tablets','Lustral 100mg tablets (Upjohn UK Ltd)','Lustral 50mg tablets (Viatris UK Healthcare Ltd)',
'Melatonin 2mg modified-release tablets','Mirtazapine 15mg orodispersible tablets','Mirtazapine 15mg tablets','Mirtazapine 15mg tablets (A A H Pharmaceuticals Ltd)','Mirtazapine 30mg orodispersible tablets',
'Mirtazapine 30mg tablets','Mirtazapine 45mg orodispersible tablets','Mirtazapine 45mg tablets','Paroxetine 10mg tablets','Paroxetine 20mg tablets','Paroxetine 30mg tablets','Paroxetine 40mg tablets',
'PAROXETINE tablets 10mg [ACTAVIS]','Propranolol 10mg tablets','Propranolol 10mg tablets (Teva UK Ltd)','Propranolol 160mg modified-release capsules','Propranolol 40mg tablets','Propranolol 40mg tablets (Teva UK Ltd)',
'Propranolol 80mg modified-release capsules','Propranolol 80mg tablets','PROPRANOLOL SR capsules 80mg [HILLCROSS]','Sertraline 100mg tablets','Sertraline 100mg tablets (A A H Pharmaceuticals Ltd)',
'Sertraline 25mg tablets','Sertraline 50mg tablets','Sertraline 50mg tablets (A A H Pharmaceuticals Ltd)','Sertraline 50mg tablets (Actavis UK Ltd)','Sertraline 50mg tablets (Teva UK Ltd)',
'Sertraline 50mg tablets (Viatris UK Healthcare Ltd)','Sertraline 50mg/5ml oral suspension','Temazepam 10mg tablets','TEXTUAL DRUG : Quetiapine 300mg tablets (Mylan)','Trazodone 100mg capsules',
'Trazodone 150mg tablets','Trazodone 150mg tablets (Zentiva)','Trazodone 50mg capsules','Trazodone 50mg capsules (A A H Pharmaceuticals Ltd)','TRAZODONE capsules 100mg [ACTAVIS]',
'Venlafaxine 150mg modified-release capsules','Venlafaxine 150mg modified-release tablets','Venlafaxine 225mg modified-release tablets','Venlafaxine 37.5mg modified-release capsules',
'Venlafaxine 37.5mg tablets','Venlafaxine 75mg modified-release capsules','Venlafaxine 75mg modified-release tablets','Venlafaxine 75mg tablets','Venlafaxine 75mg tablets (Teva UK Ltd)',
'VENLAFAXINE modified release capsules 75mg [HILLCROSS]','Venlalic XL 75mg tablets (Ethypharm UK Ltd)','Vensir XL 150mg capsules (Morningside Healthcare Ltd)',
'Vensir XL 225mg capsules (Morningside Healthcare Ltd)','Zolpidem 10mg tablets','Zolpidem 5mg tablets','Zopiclone 3.75mg tablets','Zopiclone 7.5mg tablets','Zopiclone 7.5mg tablets (A A H Pharmaceuticals Ltd)')THEN 'yes' ELSE 'no' END AS PMH_medication

FROM `yhcr-prd-phm-bia-core.CB_MYSPACE_ED.pregnancy1_pri`; 


##cleaning up
create or replace table  `yhcr-prd-phm-bia-core.CB_MYSPACE_ED.pregnancy_final1`
AS 
SELECT 
person_id
, child_person_id
, postnatal_date1
,antenatal_date
,mother_age
,event_year
,event_period
,PMH
,PMH_screening
,PMH_problems
, PMH_mangment
, PMH_care
, Depression
,Anxiety
, PMH_others
,Family_history
,pmh_medication
,ctv3code
, ctv3text
FROM `yhcr-prd-phm-bia-core.CB_MYSPACE_ED.pregnancy_final1`;

#remove the duplicate 
create or replace table  `yhcr-prd-phm-bia-core.CB_MYSPACE_ED.pregnancy_final1`
AS 
 select distinct * from `yhcr-prd-phm-bia-core.CB_MYSPACE_ED.pregnancy_final1`;

 ## select only relevant varribles

  create or replace table  `yhcr-prd-phm-bia-core.CB_MYSPACE_ED.pmh`
AS 
SELECT DISTINCT
person_id
, child_person_id
,mother_age
,event_year
,event_period
,PMH
,PMH_screening
,PMH_problems
, PMH_mangment
, PMH_care
, Depression
,Anxiety
, PMH_others
,Family_history
,pmh_medication

FROM `yhcr-prd-phm-bia-core.CB_MYSPACE_ED.pregnancy_final1`;

#remove the duplicate 
create or replace table  `yhcr-prd-phm-bia-core.CB_MYSPACE_ED.pmh`
AS 
 select distinct * from `yhcr-prd-phm-bia-core.CB_MYSPACE_ED.pmh`;

####Speaking english
create or replace table `yhcr-prd-phm-bia-core.CB_MYSPACE_ED.lan`
   AS
   SELECT 
person_id as person_id
, languagespoken as languagespoken
, speaksenglish as speaksenglish

FROM `yhcr-prd-phm-bia-core.CB_FDM_PrimaryCare_V8.tbl_srpatient`;



### joint with the baby race table

create or replace table `yhcr-prd-phm-bia-core.CB_MYSPACE_ED.lan1`
as 
SELECT 
a.person_id as person_id
,b.languagespoken as languagespoken
, b.speaksenglish as speaksenglish
FROM `yhcr-prd-phm-bia-core.CB_MYSPACE_ED.mother_baby_race`a
LEFT Join `yhcr-prd-phm-bia-core.CB_MYSPACE_ED.lan`b
on a.person_id=b.person_id
WHERE a.person_id is not null;
   ###remove the duplicate 
create or replace table  `yhcr-prd-phm-bia-core.CB_MYSPACE_ED.lan1`
AS 
 select distinct * from `yhcr-prd-phm-bia-core.CB_MYSPACE_ED.lan1`;


 


                ########index of multiple depreivation ########
 create or replace table `yhcr-prd-phm-bia-core.CB_MYSPACE_ED.IMD`
   AS
   SELECT 
LSOA_code as soa
, Index_of_Multiple_Deprivation_Decile as IMD

FROM `yhcr-prd-phm-bia-core.CB_LOOKUPS.tbl_IMD_by_LSOA`;



create or replace table `yhcr-prd-phm-bia-core.CB_MYSPACE_ED.soaa`
   AS
   SELECT 
person_id as person_id
,soa as soa


FROM `yhcr-prd-phm-bia-core.CB_FDM_PrimaryCare_V8.tbl_srpatient`;

##joining the two files

create or replace table `yhcr-prd-phm-bia-core.CB_MYSPACE_ED.IMDa1`
as select 
a.person_id as person_id
,a.soa as soa
,b.IMD as IMD

FROM `yhcr-prd-phm-bia-core.CB_MYSPACE_ED.soaa`a
left join `yhcr-prd-phm-bia-core.CB_MYSPACE_ED.IMD` b
on a.soa= b.soa 
where a.person_id is not null; 


##joint with the maternal date


 create or replace table `yhcr-prd-phm-bia-core.CB_MYSPACE_ED.pmh_IMD1`
as 
SELECT 
a.person_id as person_id

, b.IMD as IMD

FROM `yhcr-prd-phm-bia-core.CB_MYSPACE_ED.mother_baby_race`a
LEFT Join `yhcr-prd-phm-bia-core.CB_MYSPACE_ED.pmh_IMDa1`b
on a.person_id= b.person_id 
WHERE a.person_id is not null;

 


 ###remove the duplicate 
create or replace table  `yhcr-prd-phm-bia-core.CB_MYSPACE_ED.pmh_IMD1`
AS 
 select distinct * from `yhcr-prd-phm-bia-core.CB_MYSPACE_ED.pmh_IMD1`;



####Interpreter_needed
create or replace table  `yhcr-prd-phm-bia-core.CB_MYSPACE_ED.Interpreter`
AS 
SELECT 
person_id as person_id

, CASE WHEN ctv3code IN ('XaI8X', 'XaF46', 'Xaeu9', 'XaLRI', 'XE0ON', 'XaI8Y', 'XaI8X', 'XaF46', 'Xaeu9', 'XaLRI', 
'XE0ON', 'XaLSi', 'XaXwj', 'XaLSz', 'XaLRy', 'XaLSg', 'XaLSv', 'XaLRG', 'XaLRT', 'XaLSh', 'XaLT0', 
'XaLRJ', 'XaeuA', 'XaF47', 'XaPA0', 'XaLSa', 'XaLSL', 'XaLRH', 'XaLRR', 'XaPJV', 'XaLRV', 'XaLSU', 
'XaLTC', 'XaF45', 'XaI8W', 'XaLTD', 'XaBJs', 'Xaci2', 'Xacoi', 'XaLRo', 'XaLSb', 'XaLSm', 'XaLSr', 
'XaPJ6' ) then 'yes' ELSE 'no' END AS Interpreter_needed

FROM `yhcr-prd-phm-bia-core.CB_MYSPACE_ED.pregnancy1_pri`; 


#remove the duplicate 
create or replace table  `yhcr-prd-phm-bia-core.CB_MYSPACE_ED.Interpreter`
AS 
 select distinct * from `yhcr-prd-phm-bia-core.CB_MYSPACE_ED.Interpreter`;








 






