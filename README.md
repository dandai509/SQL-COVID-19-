# SQL-COVID-19-Data Analysis

## SQL Projects
### Covid 19 Data Exploration
**Code:** [`COVID 19 SQL Project.sql`](https://github.com/dandai509/Data-Analysis-Portfolio/blob/main/COVID%2019%20SQL%20Project.sql)

**Goal:** To perform EDA on COVID 19 date to show New Zealand VS World

**Description:** This project focused on Data Exploration Analysis for COVID-19 of different countries from Feb 2020 to April 2021. The dataset contains records of Covid-19 cases, deaths and vaccine records by country. Steps includes data loading, data cleaning and preprocessing and EDA 

**Skills:** Joins, CTE's, Temp Tables, Windows Functions, Aggregate Functions, Creating Views, Converting Data Types

**Technology:** SQL Server

--COVID 19 Data Exploration

--Check two tables to see if Data has imported correctly 

SELECT * 
FROM SQLPROJECT.dbo.CovidDeaths

SELECT *
FROM SQLPROJECT.dbo.CovidVaccinations

--Check overall new cases per day since Feb 2020 for each coutry 

SELECT Location, date, total_cases, new_cases, total_deaths,population
FROM SQLPROJECT.dbo.CovidDeaths
ORDER BY 1,2

-- Check Total Cases vs Total Deaths in NZ
-- The table shows the chance of dying if people contract COVID-19 in NZ

SELECT Location, date, total_cases, total_deaths,(total_deaths/total_cases)*100 as DeathPercentage
FROM SQLPROJECT.dbo.CovidDeaths
WHERE LOCATION = 'NEW ZEALAND'
ORDER BY 1,2

--Check Total cases vs Population in NZ
--The table shows COVID infection rate in NZ from Feb 2020- April 2021

SELECT Location, date, total_cases, Population,(total_cases/Population)*100 as InfectionRate
FROM SQLPROJECT.dbo.CovidDeaths
WHERE LOCATION = 'NEW ZEALAND'
ORDER BY 1,2

--Check the Highest Infection rate as per coutry
--NZ Ranked 179, one of the lowest infection rate among developed countries 

SELECT Location, MAX(total_cases) AS HighestInfectionCount, Population, MAX((total_cases/Population))*100 as InfectionRate
FROM SQLPROJECT.dbo.CovidDeaths
GROUP BY Location, Population
ORDER BY InfectionRate DESC

--Check The Highest Death Count per population as Continent
--Europe has the highest

SELECT continent, MAX(cast(Total_deaths AS int)) AS TotalDeathCount
FROM SQLPROJECT.dbo.CovidDeaths
WHERE continent is NOT NULL
GROUP BY continent
ORDER BY TotalDeathCount DESC

--Check Overall World COVID cases and death by dates 

SELECT date, SUM(new_cases) as total_cases, SUM(cast(new_deaths as int)) as total_death,SUM(cast(new_deaths as int))/SUM(new_cases)*100 as DeathPercentage
FROM SQLPROJECT.dbo.CovidDeaths
WHERE continent is not null
Group By date
order by 1, 2

--Join COVID death table and Covid Vaccination table together 

SELECT * 
FROM SQLPROJECT.dbo.CovidDeaths dea
Join SQLPROJECT.dbo.CovidVaccinations vac
	on dea.location=vac.location
	and dea.date=vac.date
	
--Check World Total Population vs Vaccination

SELECT dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations
FROM SQLPROJECT.dbo.CovidDeaths dea
Join SQLPROJECT.dbo.CovidVaccinations vac
	on dea.location=vac.location
	and dea.date=vac.date
WHERE dea.continent is not null
ORDER BY 2,3


SELECT dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations, 
SUM(CONVERT(int, vac.new_vaccinations)) OVER (PARTITION by dea.Location, dea.Date) AS RollingPeopleVaccinate

FROM SQLPROJECT.dbo.CovidDeaths dea
Join SQLPROJECT.dbo.CovidVaccinations vac
	on dea.location=vac.location
	and dea.date=vac.date
WHERE dea.continent is not null
ORDER BY 2,3

--Using CTE to perform calculation on partition by previous query

WITH PopvsVac(Continent,Location,Date,Population, new_Vaccination, RollingPeopleVaccinated)
AS
(
SELECT dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations, 
SUM(CONVERT(int, vac.new_vaccinations)) OVER (PARTITION by dea.Location, dea.Date) AS RollingPeopleVaccinated

FROM SQLPROJECT.dbo.CovidDeaths dea
Join SQLPROJECT.dbo.CovidVaccinations vac
	on dea.location=vac.location
	and dea.date=vac.date
WHERE dea.continent is not null
)
SELECT*, (RollingPeopleVaccinated/Population)*100
FROM PopvsVac

-- Using TEMP TABLE to perform calculation on Partition By in previous query
DROP TABLE IF exists #PercentPopulationVaccinated
CREATE TABLE #PercentPopulationVaccinated
( Continent nvarchar (255),
Location nvarchar(255),
Date datetime,
Population numeric,
New_vaccinations numeric,
RollingPeopleVaccinated numeric)
INSERT INTO #PercentPopulationVaccinated
SELECT dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations, 
SUM(CONVERT(int, vac.new_vaccinations)) OVER (PARTITION by dea.Location Order by dea.location, dea.Date) AS RollingPeopleVaccinated

FROM SQLPROJECT.dbo.CovidDeaths dea
Join SQLPROJECT.dbo.CovidVaccinations vac
	on dea.location=vac.location
	and dea.date=vac.date
WHERE dea.continent is not null

SELECT*, (RollingPeopleVaccinated/Population)*100
FROM #PercentPopulationVaccinated

--Creating View to store data for later visualisations

CREATE VIEW PercentPopulationVaccinated AS
SELECT dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations, 
SUM(CONVERT(int, vac.new_vaccinations)) OVER (PARTITION by dea.Location Order by dea.location, dea.Date) AS RollingPeopleVaccinated

FROM SQLPROJECT.dbo.CovidDeaths dea
Join SQLPROJECT.dbo.CovidVaccinations vac
	on dea.location=vac.location
	and dea.date=vac.date
WHERE dea.continent is not null
