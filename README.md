# SQL-PortfolioProject-1
## AlexTheAnalyst Bootcamp

**DATA EXPLORATION**
*Data Source is:
https://ourworldindata.org/covid-deaths*


``` sql
-- I had a problem with importing CSV file into Microsoft SQL SMS. I have imported file as a flat file from Tasks.
  Checked the boxes that the columns can have Null values.--


SELECT *
FROM PortfolioProject..CovidDeaths
ORDER BY 3,4

--SELECT *
--FROM PortfolioProject..CovidVaccinations
--ORDER BY 3,4


-- select data that we are going to use

SELECT location, date, total_cases, new_cases, total_deaths, population
FROM PortfolioProject..CovidDeaths
ORDER BY 1,2


-- Total Cases vs Total Deaths
-- Percentage of dying in my country from COVID
-- Here in division the result showed either Null or zero. I have found a solution to multiply column names 'Total_deaths' and 'Total_cases' by 1.0.
  After this the division worked and showed the right answers

SELECT location, date, total_cases, total_deaths, (total_deaths*1.0/total_cases*1.0)*100 AS DeathPercentage
FROM PortfolioProject..CovidDeaths
WHERE location like 'Azerbaijan'
ORDER BY 1,2

-- Total cases vs Population and Percentage of Cases

SELECT location, date, population, total_cases, (total_cases*1.0/population*1.0)*100 AS CasesPercentage
FROM PortfolioProject..CovidDeaths
ORDER BY 1,2

-- Highest Infection rate compared Population

SELECT location, 
		population, 
		MAX(total_cases) AS HighestInfectionCount, 
		MAX((total_cases*1.0/population*1.0))*100 AS PercentPopulationInfected
FROM PortfolioProject..CovidDeaths
GROUP BY location, population
ORDER BY PercentPopulationInfected DESC



-- Countries with the highest death rate
-- In report highest death rates showed the continents. Because we are preparing the reports for countries, we must exclude the continents. --

SELECT location, 
		MAX(total_deaths) AS HighestDeathCount
FROM PortfolioProject..CovidDeaths
WHERE continent is not NULL
GROUP BY location
ORDER BY HighestDeathCount DESC


-- Continents with the highest death rate per population

SELECT location,
		population,
		MAX(Total_Deaths*1.0/population*1.0)*100 AS DeathRatePopulation
FROM PortfolioProject..CovidDeaths
WHERE continent is null
GROUP BY location, population
ORDER BY DeathRatePopulation DESC

-- Global Numbers
-- here i have excluded the NULL values for new_cases. As it gave the error while dividing in zero.--

SELECT date, 
		SUM(new_cases) AS Total_Cases,
		SUM(cast(new_deaths as int)) AS Total_Deaths,
		(SUM(cast(new_deaths as int))*1.0/SUM(new_cases)*1.0)*100 AS DeathPecentage
FROM PortfolioProject..CovidDeaths
WHERE continent is not NULL
GROUP BY date
HAVING SUM(new_cases)*1.0 > 0
ORDER BY 1,2

-- Across the world

SELECT 	SUM(new_cases) AS Total_Cases,
		SUM(cast(new_deaths as int)) AS Total_Deaths,
		(SUM(cast(new_deaths as int))*1.0/SUM(new_cases)*1.0)*100 AS DeathPecentage
FROM PortfolioProject..CovidDeaths
WHERE continent is not NULL
HAVING SUM(new_cases)*1.0 > 0
ORDER BY 1,2


--JOINS

SELECT *
FROM PortfolioProject..CovidDeaths CovDea
JOIN PortfolioProject..CovidVaccinations CovVac
	ON CovDea.location = CovVac.location
	AND CovDea.date = CovVac.date


--Looking at total population vs vaccinations

SELECT CovDea.continent,
		CovDea.location,
		CovDea.date,
		CovDea.population,
		CovVac.new_vaccinations

FROM PortfolioProject..CovidDeaths CovDea
JOIN PortfolioProject..CovidVaccinations CovVac
	ON CovDea.location = CovVac.location
	AND CovDea.date = CovVac.date
WHERE CovDea.continent is not NULL
ORDER BY 2,3


-- Vacations per location

SELECT CovDea.continent,
		CovDea.location,
		CovDea.date,
		CovDea.population,
		CovVac.new_vaccinations,
		SUM(CAST(CovVac.new_vaccinations AS int)) OVER (PARTITION BY CovDea.Location
														ORDER BY CovDea.Location) AS RollingPeopleVaccinated

FROM PortfolioProject..CovidDeaths CovDea
JOIN PortfolioProject..CovidVaccinations CovVac
	ON CovDea.location = CovVac.location
	AND CovDea.date = CovVac.date
WHERE CovDea.continent is not NULL
ORDER BY 2,3


-- Population VS Vaccination

WITH PopvsVac (Continent,
				location,
				Date,
				population,
				new_vaccinations,
				RollingPeopleVaccinated)
AS
(SELECT CovDea.continent,
		CovDea.location,
		CovDea.date,
		CovDea.population,
		CovVac.new_vaccinations,
		SUM(CAST(CovVac.new_vaccinations AS int)) OVER (PARTITION BY CovDea.Location
														ORDER BY CovDea.Location, 
																 CovDea.Date) AS RollingPeopleVaccinated

FROM PortfolioProject..CovidDeaths CovDea
JOIN PortfolioProject..CovidVaccinations CovVac
	ON CovDea.location = CovVac.location
	AND CovDea.date = CovVac.date
WHERE CovDea.continent is not NULL
--ORDER BY 2,3
)
SELECT *, (RollingPeopleVaccinated/population)*100
FROM PopvsVac



--Temp Table

DROP TABLE IF Exists #PercentPopulationVaccinated
CREATE TABLE #PercentPopulationVaccinated
(
Continent nvarchar(255),
location nvarchar(255),
date datetime,
population numeric,
new_vaccinations numeric,
RollingPeopleVaccinated numeric
)

INSERT INTO #PercentPopulationVaccinated
SELECT CovDea.continent,
		CovDea.location,
		CovDea.date,
		CovDea.population,
		Convert(bigint, CovVac.new_vaccinations),
		SUM(Convert(bigint, CovVac.new_vaccinations)) OVER (PARTITION BY CovDea.Location
														ORDER BY CovDea.Location, 
																 CovDea.Date) AS RollingPeopleVaccinated

FROM PortfolioProject..CovidDeaths CovDea
JOIN PortfolioProject..CovidVaccinations CovVac
	ON CovDea.location = CovVac.location
	AND CovDea.date = CovVac.date
--WHERE CovDea.continent is not NULL
--ORDER BY 2,3

SELECT *, (RollingPeopleVaccinated/population)*100
FROM #PercentPopulationVaccinated


--Creating view to store data for later visualizations

CREATE VIEW PercentPopulationVaccinated AS
SELECT CovDea.continent,
		CovDea.location,
		CovDea.date,
		CovDea.population,
		CovVac.new_vaccinations,
		SUM(Convert(bigint, CovVac.new_vaccinations)) OVER (PARTITION BY CovDea.Location
														ORDER BY CovDea.Location, 
																 CovDea.Date) AS RollingPeopleVaccinated

FROM PortfolioProject..CovidDeaths CovDea
JOIN PortfolioProject..CovidVaccinations CovVac
	ON CovDea.location = CovVac.location
	AND CovDea.date = CovVac.date
WHERE CovDea.continent is not NULL
--ORDER BY 2,3


SELECT *
FROM PercentPopulationVaccinated
```
