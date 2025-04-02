# The SQL Murder Mystery

![image](https://github.com/user-attachments/assets/c29ed98a-31c0-4a59-b84f-96a2942a06ab)

A Walkthrough of Solution to [The SQL Murder Mystery](https://mystery.knightlab.com/)

> [!NOTE]
> A crime has taken place and the detective needs your help. The detective gave you the crime scene report, but you somehow lost it. You vaguely remember that the crime was a ​murder​ that occurred sometime on ​Jan.15, 2018​ and that it took place in ​SQL City​. Start by retrieving the corresponding crime scene report from the police department’s database.

## Crime Report
```SQL
SELECT *
FROM crime_scene_report
WHERE type = 'murder'
AND LOWER(city) = 'sql city'
AND DATE = 20180115;
```

| Date       | Type   | Description                                                                                                   | City     |
|------------|--------|--------------------------------------------------------------------------------------------------------------|----------|
| 2018-01-15 | Murder | Security footage shows that there were 2 witnesses. The first witness lives at the last house on "Northwestern Dr". The second witness, named Annabel, lives somewhere on "Franklin Ave". | SQL City |


> Security footage shows that there were 2 witnesses.
The first witness lives at the last house on "Northwestern Dr".
The second witness, named Annabel, lives somewhere on "Franklin Ave".

## Witness Statment

### Witness 1
```SQL
SELECT *
FROM person
WHERE address_street_name = 'Northwestern Dr'
ORDER BY address_number DESC
LIMIT 1;
```

| ID    | Name           | License ID | Address Number | Address Street Name | SSN        |
|-------|---------------|------------|----------------|----------------------|------------|
| 14887 | Morty Schapiro | 118009     | 4919           | Northwestern Dr      | 111-56-4949 |

### Witness 2

```SQL
SELECT *
FROM person
WHERE address_street_name = 'Franklin Ave'
AND name LIKE 'Annabel%';
```

| ID    | Name           | License ID | Address Number | Address Street Name | SSN         |
|-------|---------------|------------|----------------|----------------------|-------------|
| 16371 | Annabel Miller | 490173     | 103            | Franklin Ave         | 318-77-1143 |

### Witness Transcripts

```SQL
SELECT *
FROM interview
WHERE person_id IN (14887, 16371);
```

| Person ID | Transcript |
|-----------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| 14887     | I heard a gunshot and then saw a man run out. He had a "Get Fit Now Gym" bag. The membership number on the bag started with "48Z". Only gold members have those bags. The man got into a car with a plate that included "H42W". |
| 16371     | I saw the murder happen, and I recognized the killer from my gym when I was working out last week on January the 9th. |

### Witness 1
> I heard a gunshot and then saw a man run out. He had a "Get Fit Now Gym" bag. The membership number on the bag started with "48Z". Only gold members have those bags. The man got into a car with a plate that included "H42W".

### Witness 2
> I saw the murder happen, and I recognized the killer from my gym when I was working out last week on January the 9th.

## Suspects

```SQL
SELECT *
FROM get_fit_now_member a
JOIN get_fit_now_check_in b
ON a.id = b.membership_id
WHERE a.membership_status = 'gold'
AND a.id LIKE '48Z%'
AND check_in_date = '20180109';
```

### Gym Membership and Check-in Data

| ID    | Person ID | Name          | Membership Start Date | Membership Status | Membership ID | Check-in Date | Check-in Time | Check-out Time |
|-------|----------|---------------|----------------------|------------------|---------------|--------------|--------------|---------------|
| 48Z7A | 28819    | Joe Germuska  | 2016-03-05           | Gold             | 48Z7A         | 2018-01-09   | 16:00        | 17:30         |
| 48Z55 | 67318    | Jeremy Bowers | 2016-01-01           | Gold             | 48Z55         | 2018-01-09   | 15:30        | 17:00         |


### Car and Suspect Details

```SQL
SELECT *
FROM drivers_license
WHERE plate_number LIKE '%H42W%'
AND gender = 'male';
```

| ID     | Age | Height (in) | Eye Color | Hair Color | Gender | Plate Number | Car Make   | Car Model    |
|--------|-----|------------|-----------|------------|--------|--------------|------------|-------------|
| 423327 | 30  | 70         | Brown     | Brown      | Male   | 0H42W2       | Chevrolet  | Spark LS    |
| 664760 | 21  | 71         | Black     | Black      | Male   | 4H42WR       | Nissan     | Altima      |


### Suspect Personal Details

```SQL
SELECT * 
FROM person
WHERE license_id IN (423327, 664760)
```

| ID     | Name           | License ID | Address Number | Address Street Name         | SSN        |
|--------|---------------|------------|----------------|-----------------------------|------------|
| 51739  | Tushar Chandra | 664760     | 312            | Phi St                      | 137-88-2671 |
| 67318  | Jeremy Bowers  | 423327     | 530            | Washington Pl, Apt 3A       | 871-53-9279 |


## Suspect Filtering

```SQL
SELECT p.id, p.name, p.license_id, gfn.id, gfn.membership_status 
FROM person p 
JOIN get_fit_now_member gfn
ON p.id = gfn.person_id
WHERE license_id IN (423327, 664760)
```

| ID     | Name          | License ID | Membership ID | Membership Status |
|--------|---------------|------------|---------------|-------------------|
| 67318  | Jeremy Bowers | 423327     | 48Z55         | Gold              |

![image](https://github.com/user-attachments/assets/1583e8e3-7036-4382-8da3-9b046906bff5)

## Further Investigation

### Murderer's Transcript
```SQL
SELECT *
FROM interview
WHERE person_id = 67318;
```

| Person ID | Transcript |
|-----------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| 67318     | I was hired by a woman with a lot of money. I don't know her name but I know she's around 5'5" (65") or 5'7" (67"). She has red hair and she drives a Tesla Model S. I know that she attended the SQL Symphony Concert 3 times in December 2017. |

> [!IMPORTANT]
> I was hired by a woman with a lot of money. I don't know her name but I know she's around 5'5" (65") or 5'7" (67"). She has red hair and she drives a Tesla Model S. I know that she attended the SQL Symphony Concert 3 times in December 2017.

### Car and Suspect Details

```SQL
SELECT *
FROM drivers_license
WHERE height BETWEEN 65 AND 67
AND hair_color = 'red'
AND car_make = 'Tesla';
```

| ID     | Age | Height (in) | Eye Color | Hair Color | Gender | Plate Number | Car Make | Car Model  |
|--------|-----|-------------|-----------|------------|--------|--------------|----------|------------|
| 202298 | 68  | 66          | Green     | Red        | Female | 500123       | Tesla    | Model S   |
| 291182 | 65  | 66          | Blue      | Red        | Female | 08CM64       | Tesla    | Model S   |
| 918773 | 48  | 65          | Black     | Red        | Female | 917UU3       | Tesla    | Model S   |

### Suspect Details

```SQL
SELECT * FROM PERSON
WHERE license_id IN (202298, 291182, 918773);
```

| ID     | Name            | License ID | Address Number | Address Street Name | SSN        |
|--------|-----------------|------------|----------------|----------------------|------------|
| 78881  | Red Korb        | 918773     | 107            | Camerata Dr          | 961-38-8910 |
| 90700  | Regina George   | 291182     | 332            | Maple Ave            | 337-16-9072 |
| 99716  | Miranda Priestly| 202298     | 1883           | Golden Ave           | 987-75-6388 |


### Event Attendance Details
```SQL
SELECT *
FROM facebook_event_checkin
WHERE person_id IN (78881, 90700, 99716)
AND event_name = 'SQL Symphony Concert'
AND date LIKE '201712%';
```

| Person ID | Event ID | Event Name              | Date       |
|-----------|----------|-------------------------|------------|
| 99716     | 1143     | SQL Symphony Concert    | 2017-12-06 |
| 99716     | 1143     | SQL Symphony Concert    | 2017-12-12 |
| 99716     | 1143     | SQL Symphony Concert    | 2017-12-29 |


## Final Suspect

```SQL
SELECT * FROM person
WHERE id = 99716;
```

| ID     | Name             | License ID | Address Number | Address Street Name | SSN        |
|--------|------------------|------------|----------------|----------------------|------------|
| 99716  | Miranda Priestly | 202298     | 1883           | Golden Ave           | 987-75-6388 |


![image](https://github.com/user-attachments/assets/61125374-f735-4365-abfd-4a46118bdcd2)










