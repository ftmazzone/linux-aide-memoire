# Conky - exemple de configuration pour debian (buster)

```ini
# ~/.config/conky/conky.conf
#
# conky.config = {
#     alignment = 'top_right',
#     background = false,
#     border_width = 1,
#     cpu_avg_samples = 2,
#     default_color = 'white',
#     default_outline_color = 'white',
#     default_shade_color = 'white',
#     draw_borders = false,
#     draw_graph_borders = true,
#     draw_outline = false,
#     draw_shades = false,
#     use_xft = true,
#     font = 'DejaVu Sans Mono:size=11',
#     gap_x = 0,
#     gap_y = 10,
#     minimum_height = 0,
#     minimum_width = 0,
#     net_avg_samples = 2,
#     no_buffers = true,
#     out_to_console = false,
#     out_to_stderr = false,
#     extra_newline = false,
#     double_buffer = true,
#     own_window_hints="undecorated,below,skip_taskbar,skip_pager,sticky",
#     own_window_transparent = false,
#     own_window = true,
#     own_window_argb_visual = true,
#     own_window_argb_value = 145,
#     own_window_class = 'desktop',
#     own_window_type = "normal",
#     stippled_borders = 0,
#     update_interval = 30.0,
#     uppercase = false,
#     use_xft = true,
#     use_spacer = 'none',
#     show_graph_scale = false,
#     show_graph_range = false,
# }
# 
# --${scroll 16 $nodename - $sysname $kernel sur $machine | }
#
# conky.text = [[
#
# $nodename - $sysname $kernel sur $machine
# $hr
# ${if_match ${execi 30 expr $(($(date +%s%N)/1000000000)) % 60} < 30}
# ${color grey}Durée de fonctionnement:$color $uptime
# ${color grey}Fréquence (GHz):$color $freq_g
# ${color grey}RAM:$color $mem/$memmax - $memperc% ${membar 4}
# ${color grey}Swap:$color $swap/$swapmax - $swapperc% ${swapbar 4}
# ${color grey}CPU:$color ${texeci 60 echo $(($(cat /sys/class/thermal/thermal_zone0/temp)/1000))}°C - $cpu% ${cpubar 4} --Température pour un Raspberry Pi
# ${color grey}Processus:$color $processes ${color grey}En cours d'exécution:$color $running_processes
# $hr
# ${color grey}Système de fichiers:
# / $color${fs_used /}/${fs_size /} ${fs_bar 6 /}
# $hr
# ${color grey}Nom                PID    CPU%   MEM%
# ${color lightgrey} ${top name 1} ${top pid 1} ${top cpu 1} ${top mem 1}
# ${color lightgrey} ${top name 2} ${top pid 2} ${top cpu 2} ${top mem 2}
# ${color lightgrey} ${top name 3} ${top pid 3} ${top cpu 3} ${top mem 3}
# ${color lightgrey} ${top name 4} ${top pid 4} ${top cpu 4} ${top mem 4}
# ${else}
# ${color lightgrey}${texeci 1800 wget -q -O - fr.wttr.in?T?n| head -37}
# ${endif}
# ]]

cat > ~/.config/conky/conky.conf <<EOL
conky.config = {
    alignment = 'top_right',
    background = false,
    border_width = 1,
    cpu_avg_samples = 2,
    default_color = 'white',
    default_outline_color = 'white',
    default_shade_color = 'white',
    draw_borders = false,
    draw_graph_borders = true,
    draw_outline = false,
    draw_shades = false,
    use_xft = true,
    font = 'DejaVu Sans Mono:size=11',
    gap_x = 0,
    gap_y = 10,
    minimum_height = 0,
    minimum_width = 0,
    net_avg_samples = 2,
    no_buffers = true,
    out_to_console = false,
    out_to_stderr = false,
    extra_newline = false,
    double_buffer = true,
    own_window_hints="undecorated,below,skip_taskbar,skip_pager,sticky",
    own_window_transparent = false,
    own_window = true,
    own_window_argb_visual = true,
    own_window_argb_value = 145,
    own_window_class = 'desktop',
    own_window_type = "normal",
    stippled_borders = 0,
    update_interval = 30.0,
    uppercase = false,
    use_xft = true,
    use_spacer = 'none',
    show_graph_scale = false,
    show_graph_range = false,
}
--\${scroll 16 \$nodename - \$sysname \$kernel sur \$machine | }
conky.text = [[
\$nodename - \$sysname \$kernel sur \$machine
\$hr
\${if_match \${execi 30 expr \$((\$(date +%s%N)/1000000000)) % 60} < 30}
\${color grey}Durée de fonctionnement:\$color \$uptime
\${color grey}Fréquence (GHz):\$color \$freq_g
\${color grey}RAM:\$color \$mem/\$memmax - \$memperc% \${membar 4}
\${color grey}Swap:\$color \$swap/\$swapmax - \$swapperc% \${swapbar 4}
\${color grey}CPU:\$color \${texeci 60 echo \$((\$(cat /sys/class/thermal/thermal_zone0/temp)/1000))}°C - \$cpu% \${cpubar 4} --Température pour un Raspberry Pi
\${color grey}Processus:\$color \$processes \${color grey}En cours d'exécution:\$color \$running_processes
\$hr
\${color grey}Système de fichiers:
 / \$color\${fs_used /}/\${fs_size /} \${fs_bar 6 /}
\$hr
\${color grey}Nom                PID    CPU%   MEM%
\${color lightgrey} \${top name 1} \${top pid 1} \${top cpu 1} \${top mem 1}
\${color lightgrey} \${top name 2} \${top pid 2} \${top cpu 2} \${top mem 2}
\${color lightgrey} \${top name 3} \${top pid 3} \${top cpu 3} \${top mem 3}
\${color lightgrey} \${top name 4} \${top pid 4} \${top cpu 4} \${top mem 4}
\${else}
\${color lightgrey}\${texeci 1800 wget -q -O - fr.wttr.in?T?n| head -37}
\${endif}
]]
EOL
```
