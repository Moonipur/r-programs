# Basic R Programs

### Visualize 1 (column plot)
```
library(tidyverse)
library(readxl)
library(ggpubr) # if don't have -> install.packages('ggpubr')

df1 = read_excel('U2OS_9_timepoint.xlsx')
df1$Treatment = factor(df1$Treatment)
df1$Time = factor(df1$Time)

my_comparisons <- list(c("Control", "MPA"), c("Control", "MPA_G"))

df1 %>% ggbarplot( 
   x = "Treatment",
   y = "Perc_cells",
   fill = "Time",
   add = c("mean_se"), # add mean and standard error
   position = position_dodge()
) +
  facet_wrap(~Time, ncol = 9, scales = "free_x") +
  stat_compare_means(comparisons = my_comparisons, method = "t.test", label = "p.signif") +
  theme(
    axis.text.x = element_text(angle = 90, hjust = 1),
    strip.background = element_blank(), 
    strip.text = element_text(face = "bold")
    ) +
  labs(y = "Mean of percent cell (%)")
```

### Visualize 2 (swimmimg plot)
```
library(tidyverse)
library(ggnewscale) # if don't have -> install.packages('ggnewscale')

df_main <- data.frame(
  ID = factor(c("003", "004", "009", "022")),
  End = c(2000, 1950, 1900, 1850),
  Status = c("Alive", "Alive", "Alive", "Deceased"),
  Recurrence_Start = c(450, 600, 1100, 800) # For the tan shading
)

df_therapy <- data.frame(
  ID = factor(c("003", "003", "004", "004")),
  Type = c("Chemotherapy", "PARPi", "Chemotherapy", "Bevacizumab"),
  Start = c(50, 600, 20, 500),
  End = c(400, 1500, 300, 1400),
  y_offset = c(0.2, -0.2, 0.2, -0.2) # To prevent overlapping lines
)

df_surgeries <- data.frame(
  ID = factor(c("003", "004", "004")),
  Type = c("Primary debulking", "Primary debulking", "Secondary debulking"),
  Time = c(10, 10, 650)
)

ggplot() +
  # Layer 1: Light gray background bars for total survival
  geom_rect(data = df_main, aes(xmin = 0, xmax = End, 
                                ymin = as.numeric(ID)-0.4, ymax = as.numeric(ID)+0.4),
            fill = "grey95") +
  
  # Layer 2: Shaded "Post-recurrence" region
  geom_rect(data = df_main, aes(xmin = Recurrence_Start, xmax = End, 
                                ymin = as.numeric(ID)-0.4, ymax = as.numeric(ID)+0.4),
            fill = "#f2e1d1", alpha = 0.8) +
  
  # Layer 3: Therapy Segments (using y_offset to stack them)
  geom_segment(data = df_therapy, aes(x = Start, xend = End, 
                                      y = as.numeric(ID) + y_offset, 
                                      yend = as.numeric(ID) + y_offset, 
                                      color = Type), size = 2) +
  
  # Layer 4: Surgeries (Points)
  geom_point(data = df_surgeries, aes(x = Time, y = as.numeric(ID), fill = Type), 
             shape = 21, size = 4, color = "white") +
  
  # Layer 5: Survival Status (Vertical line at the end)
  geom_segment(data = df_main, aes(x = End, xend = End, 
                                   y = as.numeric(ID)-0.4, yend = as.numeric(ID)+0.4,
                                   color = Status), size = 3) +
  
  # Styling
  scale_y_continuous(breaks = 1:length(levels(df_main$ID)), labels = levels(df_main$ID)) +
  scale_color_manual(values = c("Chemotherapy" = "#ff7f0e", "PARPi" = "#2ca02c", "Bevacizumab" = "purple"), name = "Therapy") +
  new_scale_color() + 
  # --------------------------

  # Layer 5: Survival Status (Now uses a "fresh" color scale)
  geom_segment(data = df_main, aes(x = End, xend = End, 
                                 y = as.numeric(ID)-0.4, yend = as.numeric(ID)+0.4,
                                 color = Status), size = 3) +
  # Define the second color scale (Survival)
  scale_color_manual(values = c("Alive" = "#a6d96a", "Deceased" = "black"), 
                     name = "Survival Status") +
  
  theme_minimal() +
  labs(x = "Time from diagnosis (days)", y = "Patient ID")

```
