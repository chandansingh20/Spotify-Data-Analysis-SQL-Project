# Spotify-Data-Analysis-SQL-Project
This project involves analyzing a Spotify dataset with various attributes about tracks, albums, and artists using SQL. It covers an end-to-end process of normalizing a denormalized dataset, performing SQL queries of varying complexity (easy, medium, and advanced), and optimizing query performance. The primary goals of the project are to practice advanced SQL skills and generate valuable insights from the dataset.

-- create table
DROP TABLE IF EXISTS spotify;
CREATE TABLE spotify (
    artist VARCHAR(255),
    track VARCHAR(255),
    album VARCHAR(255),
    album_type VARCHAR(50),
    danceability FLOAT,
    energy FLOAT,
    loudness FLOAT,
    speechiness FLOAT,
    acousticness FLOAT,
    instrumentalness FLOAT,
    liveness FLOAT,
    valence FLOAT,
    tempo FLOAT,
    duration_min FLOAT,
    title VARCHAR(255),
    channel VARCHAR(255),
    views FLOAT,
    likes BIGINT,
    comments BIGINT,
    licensed BOOLEAN,
    official_video BOOLEAN,
    stream BIGINT,
    energy_liveness FLOAT,
    most_played_on VARCHAR(50)
);
Project Steps
1. Data Exploration
Before diving into SQL, it’s important to understand the dataset thoroughly. The dataset contains attributes such as:

Artist: The performer of the track.
Track: The name of the song.
Album: The album to which the track belongs.
Album_type: The type of album (e.g., single or album).
Various metrics such as danceability, energy, loudness, tempo, and more.
4. Querying the Data
After the data is inserted, various SQL queries can be written to explore and analyze the data. Queries are categorized into easy, medium, and advanced levels to help progressively develop SQL proficiency.

SELECT * FROM public.spotify
LIMIT 100

--- EDA

SELECT COUNT(*) FROM spotify;

SELECT COUNT(DISTINCT artist) FROM spotify;

SELECT COUNT(DISTINCT album) FROM spotify;

SELECT DISTINCT album_type FROM spotify;

SELECT max(duration_min) FROM spotify;

SELECT min(duration_min) FROM spotify;

SELECT * from spotify
WHERE duration_min = 0;

delete from spotify
WHERE duration_min = 0;

SELECT * from spotify
WHERE duration_min = 0;

select distinct channel FROM spotify;

SELECT distinct most_played_on from spotify


--------------------------------
-- data analysis - easy category
--------------------------------

--1.Retrieve the names of all tracks that have more than 1 billion streams.

SELECT track from spotify
where stream > 1000000000


--2.List all albums along with their respective artists.

select 
	distinct album,
	artist
from spotify
order by 1


--3.Get the total number of comments for tracks where licensed = TRUE.

SELECT 
	sum(comments) as total_comments 
from spotify
where licensed = 'true'


--4.Find all tracks that belong to the album type single.

select
	track
from spotify
where album_type = 'single'


--5.Count the total number of tracks by each artist.

select
	artist,
	count(track) as total_no_of_tracks
from spotify
group by artist
order by 2 desc



----------------------------------
-- data analysis - medium category
----------------------------------

--1.Calculate the average danceability of tracks in each album.

SELECT
	album,
	avg(danceability) as avg_danceability
FROM spotify
group by album
order by avg_danceability desc

--2.Find the top 5 tracks with the highest energy values.

SELECT
	track,
	max(energy)
FROM spotify
group by 1
order by 2 DESC
LIMIT 5

--3.List all tracks along with their views and likes where official_video = TRUE.

SELECT
	track,
	sum(views) as total_views,
	sum(likes) as total_likes
FROM spotify
WHERE official_video = 'true'
group by 1
order by 2 desc

--4.For each album, calculate the total views of all associated tracks.

SELECT
	album,
	track,
	sum(views) as total_views
FROM spotify
group by 1,2
order by 3 desc

--5.Retrieve the track names that have been streamed on Spotify more than YouTube.

WITH streamed_track AS
(
SELECT
	track,
	coalesce(sum(case when most_played_on = 'Youtube' then stream end),0) as streamed_on_youtube,
	coalesce(sum(case when most_played_on = 'Spotify' then stream end),0) as streamed_on_spotify
FROM spotify
group by 1
)
SELECT * from streamed_track
WHERE 
	streamed_on_spotify > streamed_on_youtube
	and
	streamed_on_youtube <> 0


------------------------------------
-- data analysis - Advanced category
------------------------------------

--1.Find the top 3 most-viewed tracks for each artist using window functions.

with ranking_artist as
(
select 
	artist,
	track,
	sum(views) as total_views,
	dense_rank() over(partition by artist order by sum(views) DESC) as drn
from spotify 
group by 1,2
order by 1,3 desc
)
select * from ranking_artist
where drn <= 3

--2.Write a query to find tracks where the liveness score is above the average.

select 
	s1.track,
	s1.liveness
from spotify as s1
where s1.liveness > (select avg(s2.liveness)
					from spotify as s2)

--3.Use a WITH clause to calculate the difference between the highest and lowest energy values for tracks in each album.

with difference as
(
select
	album,
	max(energy) as highest_energy,
	min(energy) as lowest_energy
from spotify
group by 1
)
select
	album,
	highest_energy-lowest_energy as energy_diff
from difference
order by 2 desc

--4.Find tracks where the energy-to-liveness ratio is greater than 1.2.

SELECT
	track
from spotify
where energy/liveness > 1.2


--5.Calculate the cumulative sum of likes for tracks ordered by the number of views, using window functions.

SELECT 
	track,
	sum(likes) over(order by views desc) as total_likes
from spotify
