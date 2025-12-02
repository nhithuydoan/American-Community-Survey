# CLAUDE.md - American Community Survey Data Analysis

## Project Overview

This is an R-based data analysis project examining American Community Survey data to analyze earnings, educational outcomes, and gender disparities across different majors and occupations from 2012-2019.

**Note**: The repository name contains a typo: "Servey" instead of "Survey"

## Repository Structure

```
American-Community-Servey/
├── Final-code.R              # Main analysis script (primary workflow)
├── datawrangling.R           # Data compilation and cleaning pipeline
├── draft.R                   # Experimental/draft data processing code
├── .Rhistory                 # R session history
├── category_names.rds        # Serialized category names reference data
├── earnings-2013-2019.csv    # Compiled earnings dataset (OUTPUT from datawrangling.R)
├── recent-grads-2012.csv     # Recent graduates data (2012)
├── grad-students-2012.csv    # Graduate students data (2012)
├── women-stem-2012.csv       # Women in STEM data (2012)
├── STEM-vs-NonSTEM2012.xlsx  # STEM classification reference
├── median-earnings-2012.xlsx # Raw earnings data (2012)
└── median-earnings-2013-final.xlsx through median-earnings-2019-final.xlsx
```

## Research Questions

The analysis addresses three main research questions:

### Research Question 1: Highest/Lowest Earning Majors
- What are the highest-earning majors?
- What are the lowest-earning majors?
- How do earnings vary between undergraduate and graduate degrees across top 10 popular graduate majors?

### Research Question 2: STEM vs Non-STEM Analysis
- Is STEM majors' median salary higher than non-STEM majors?
- Is non-STEM majors' unemployment rate higher than STEM majors?

### Research Question 3: Gender and Earnings
- How popular is each major category across both genders?
- Do men major in STEM more than women?
- How do male/female dominated majors relate to median earnings?
- Gender pay gap analysis across earning groups (top, middle, bottom)
- How do earning differences evolve over years (2013-2019)?

## Data Workflow

### 1. Data Wrangling (`datawrangling.R`)

**Purpose**: Compiles and cleans multiple Excel files into a unified dataset

**Key Functions**:

- `compile(filename, input_year)` - For years 2013-2017
  - Reads Excel files with custom structure (skip first 7 rows)
  - Removes margin of error columns
  - Standardizes column names
  - Categorizes occupations into major/minor categories
  - Filters out header rows and summary rows

- `compile2(filename, input_year)` - For years 2018-2019
  - Similar to `compile()` but handles slightly different format
  - Adjusts for different row counts and occupation label formatting

**Output**: `earnings-2013-2019.csv` with columns:
- year, occupation, major, TotalPopulation, Number_of_Men, Number_of_Women
- Women_Percentage, Earning, Men_earning, Women_earning, Compare_to_men

### 2. Analysis (`Final-code.R`)

**Purpose**: Main analysis pipeline with data exploration, visualization, and statistical modeling

**Workflow Stages**:

1. **Data Loading** (lines 17-24)
   - Loads CSV files and RDS reference data
   - Converts numeric columns
   - Removes index column

2. **Initial Data Exploration** (lines 31-112)
   - Creates derived variables for graduate vs undergraduate comparison
   - Creates STEM vs Non-STEM classification
   - Calculates means and aggregates

3. **STEM Classification** (lines 77-83)
   - Non-STEM: Natural Resources/Construction/Maintenance, Management/Business/Financial, Education/Legal/Community Service/Arts/Media, Service, Sales/Office, Production/Transportation/Material Moving
   - STEM: Healthcare Practitioners/Technical, Computer/Engineering/Science
   - Special case: Sales engineers classified as STEM

4. **Data Transformations**:
   - Gender ratio calculations
   - Earning group stratification (top 10, middle 10, bottom 10 occupations)
   - Longitudinal analysis across years

5. **Visualizations** (9+ figures):
   - Figure 1: Top 10 highest earning majors
   - Figure 2: Top 10 lowest earning majors
   - Figure 3: Graduate vs undergraduate median salary
   - Figures 4-5: STEM vs Non-STEM unemployment and salary comparison
   - Figure 6: Major categories across genders
   - Figure 7: Earnings and gender ratio correlation
   - Figures 8-11: Gender pay gap analysis, STEM field analysis, temporal trends

6. **Statistical Modeling** (lines 247-350):
   - **STEM Effect Model**: `lm(Earning ~ STEM)`
   - **Gender Effect Models**: `lm(Earning_both ~ Gender)` for top/middle/bottom earning groups
   - **Combined Model**: `lm(Earning_both ~ Gender + STEM)`
   - **Interaction Model**: `lm(Earning_both ~ Gender + STEM + Gender*STEM)`

## Key R Libraries

### Core Dependencies
- `tidyverse` - Data manipulation and visualization ecosystem
- `dplyr` - Data manipulation
- `tidyr` - Data tidying (gather, spread operations)
- `ggplot2` - Data visualization
- `forcats` - Factor manipulation

### Visualization
- `RColorBrewer` - Color palettes
- `ggpubr` - Publication-ready plots
- `ggExtra` - Additional ggplot2 features
- `corrplot` - Correlation matrix visualization

### Data Import/Export
- `readxl` - Reading Excel files
- `openxlsx` - Writing Excel files

### Other
- `mdsr` - Modern Data Science with R utilities
- `dbplyr` - Database backend for dplyr

## Coding Conventions

### Data Manipulation Patterns

1. **Pipeline Style**: Heavy use of `%>%` (pipe operator) for readable data transformations
   ```r
   data %>%
     filter(condition) %>%
     select(columns) %>%
     mutate(new_var = calculation) %>%
     arrange(desc(column))
   ```

2. **Variable Naming**:
   - Snake_case for data frame columns: `Major_category`, `Median_salary`
   - Dot notation for intermediate objects: `recent.grads.students`, `higher.degrees`
   - Descriptive names: `earning_2013_topcareer`, `stem.med.salary`

3. **Data Reshaping**:
   - `gather()` - Wide to long format (e.g., converting Men_earning/Women_earning to single Gender/Earning columns)
   - `spread()` - Long to wide format (less common in this codebase)

4. **Factor Handling**:
   - `fct_reorder()` - Reorder factors by another variable (for ordered visualizations)
   - `as.factor()` - Explicit factor conversion
   - `levels()` - Rename factor levels for cleaner labels

### Visualization Conventions

1. **ggplot2 Structure**:
   ```r
   ggplot(data, aes(x = var1, y = var2, color = var3)) +
     geom_point() / geom_col() / geom_bar() +
     scale_color_brewer(palette = "Set1") +
     coord_flip() +  # For horizontal bar charts
     theme_bw() / theme_classic() +
     labs(title = "Figure X: TITLE", subtitle = "context", x = "", y = "")
   ```

2. **Color Palettes**: ColorBrewer palettes (Set1, Set2, Paired, Dark2, YlOrRd)

3. **Figure Numbering**: Sequential numbering with descriptive ALL CAPS titles

4. **Common Geoms**:
   - `geom_col()` - Bar charts for aggregated data
   - `geom_point()` - Scatter plots
   - `geom_jitter()` - Jittered points to reduce overplotting
   - `geom_errorbar()` - For showing ranges (25th to 75th percentile)
   - `geom_smooth(method="lm")` - Linear regression lines
   - `geom_line()` - Connecting points or trends

### Statistical Analysis Conventions

1. **Linear Models**:
   - Use `lm()` for regression analysis
   - Always follow with `summary()` to examine results
   - Use `anova()` for variance analysis
   - Compare nested models to assess added variable significance

2. **Subsetting**:
   - `subset()` for row filtering with conditions
   - `filter()` from dplyr for pipeline-style filtering
   - `select()` for column selection

3. **Missing Data**:
   - Use `na.omit()` when calculating means
   - Filter with `complete.cases()` for model data

### Data Preparation Patterns

1. **Year-wise Processing**:
   - Create function for repeated operations across years
   - Store results in year-specific objects (`earning_2013`, `earning_2014`, etc.)
   - Combine using `rbind()` or `bind_rows()`

2. **Group Stratification**:
   - Top earners: rows 1-10
   - Middle earners: rows 251-260 (approximately row 250 out of ~520)
   - Bottom earners: rows 513-522

3. **Column Naming**:
   - Use `colnames()` for explicit renaming
   - Use `make.names()` for creating valid R names
   - Consistent naming after each major transformation

## Development Guidelines for AI Assistants

### When Modifying Code

1. **Preserve Existing Structure**:
   - Maintain the three-part structure: Data Loading → Exploration → Visualization/Modeling
   - Keep figure numbers sequential
   - Preserve existing variable names unless refactoring

2. **Adding New Analysis**:
   - Follow the established pattern: data preparation → visualization → statistical test
   - Create descriptive figure titles with ALL CAPS convention
   - Use ColorBrewer palettes for consistency

3. **Data Wrangling**:
   - If modifying `datawrangling.R`, ensure output format matches `earnings-2013-2019.csv` schema
   - Test with one year first, then apply to all years
   - Update both `compile()` and `compile2()` if changing core logic

4. **Testing Changes**:
   - Source the script in R to check for syntax errors
   - Verify visualizations render correctly
   - Check model summaries for statistical significance
   - Ensure no breaking changes to downstream analyses

### Common Tasks

**Adding a New Year of Data**:
1. Add new Excel file: `median-earnings-YYYY-final.xlsx`
2. Update `datawrangling.R`: Add `earning_YYYY <- compile2("median-earnings-YYYY-final.xlsx", YYYY)`
3. Update `rbind()` call to include new year
4. Re-run to regenerate `earnings-2013-2019.csv` (rename if needed)
5. Update `Final-code.R` if year-specific analysis needed

**Adding New Visualizations**:
1. Prepare data subset/transformation
2. Create ggplot with consistent styling
3. Assign sequential figure number
4. Use descriptive ALL CAPS title
5. Save to object: `figureN <- ggplot(...)`
6. Call object name to render: `figureN`

**Modifying STEM Classification**:
1. Update lines 77-83 in `Final-code.R`
2. Verify classification logic covers all majors
3. Re-run dependent analyses (figures 8-11, models 2-4)

**Adding Statistical Tests**:
1. Prepare clean dataset (handle NA values)
2. Create model: `modelN <- lm(y ~ x + ...)`
3. Examine: `summary(modelN)`
4. Create corresponding visualization
5. Document findings in comments

### Important Notes

1. **Scientific Notation**: `options(scipen=999)` disables scientific notation for better readability

2. **Data Dependencies**:
   - `earnings-2013-2019.csv` is generated by `datawrangling.R`
   - `category_names.rds` is a pre-existing reference file
   - Excel files are raw input data

3. **Reproducibility**:
   - All data files must be present in working directory
   - Run `datawrangling.R` before `Final-code.R` if Excel files are updated
   - No absolute file paths - relies on working directory

4. **Known Issues**:
   - Duplicate file: `median-earnings-2014-finall.xlsx` (three l's) - likely a typo
   - Mixed terminology: "Median" vs "Earning" vs "Salary" used interchangeably
   - Some commented-out code (lines 22, 66-69) - kept for reference

5. **Performance**:
   - Dataset size is manageable (~3600 rows in combined data)
   - No major performance concerns
   - All operations complete in reasonable time

## Git Workflow

- Current branch: Feature branches with `claude/` prefix
- Main branch: Typically `main` or `master`
- Always commit logical units of work with descriptive messages
- Push to feature branch before creating pull requests

## Additional Resources

Referenced in code comments:
- https://github.com/rfordatascience/tidytuesday/blob/master/data/2019/2019-03-05/jobs_gender.csv

## Quick Start for AI Assistants

1. **To reproduce analysis**:
   ```r
   source("datawrangling.R")  # Only if Excel files changed
   source("Final-code.R")      # Main analysis
   ```

2. **To explore data structure**:
   ```r
   str(recent.grads.students)
   head(earning_2013)
   summary(data)
   ```

3. **To modify visualizations**:
   - Locate figure code in `Final-code.R`
   - Modify ggplot layers
   - Re-run the specific figure code block

4. **To add new statistical tests**:
   - Follow pattern in lines 247-350
   - Prepare data, create model, summarize results
   - Create supporting visualization

---

**Last Updated**: 2025-12-02
**Primary Language**: R
**Analysis Period**: 2012-2019
**Data Source**: American Community Survey (Census Bureau)
