// This source code is subject to the terms of the Mozilla Public License 2.0 at https://mozilla.org/MPL/2.0/
// © boitoki
//
// 🛟 https://boitoki.gitbook.io/htf-candles-and-pivots/
// =====================================================
//@version=5
indicator('HTF Candles & Pivots', 'HTF', overlay=true, max_lines_count=300, max_labels_count=100, max_boxes_count=300, max_bars_back=1000)

import boitoki/AwesomeColor/4 as ac
import boitoki/Pivots/5 as f

//////////////////////
// Define
//////////////////////
GP1                 = 'Ϸ𐊦ѴΘ𐨝Ⴝ'
GP2                 = '𐋉𐊤𐑿𐊤𐊯𐋎𐑖'
GP3                 = 'ꡘ𐨝𐊥 𐊢𐋎𐑿ᱚ𐑖𐊤'
GP4                 = ''
GP5                 = 'ѴΘ𐑖Ϥ𐊰𐊤 Ϸ𐑖Θ𐊥𐊦𐑖𐊤'
GP6                 = '𐊤𐋂Ϸ𐊤𐊢𐨝𐊤ᱚ Ϸ𐊯𐊦𐊢𐊤 𐊯𐋎𐑿𐋉𐊤'
GP7                 = 'ΘϷ𐨝𐊦Θ𐑿Ⴝ'
GP8                 = '𐑖𐋎𐌱𐊤𐑖'
GP9                 = '𐊤𐋂Ϸ𐊤𐊯𐊦𐊰𐊤𐑿𐨝𐋎𐑖'

t_PP                = 'PP'
t_R5                = 'R ⁵'
t_R4                = 'R ⁴'
t_R3                = 'R ³'
t_R2                = 'R ²'
t_R1                = 'R ¹'
t_S1                = 'S ¹'
t_S2                = 'S ²'
t_S3                = 'S ³'
t_S4                = 'S ⁴'
t_S5                = 'S ⁵'
t_BC                = 'BC'
t_TC                = 'TC'

f_price             = '{0,number,#.#####}'
f_price_shr         = '{0,number,#.###}'
f_pip               = '{0,number,#.#} ᵖⁱᵖᔆ'
f_pip_              = '{0,number,#.#}'

maximum_x           = bar_index + 500
separator_dot       = ' • '
separator_br        = '\n'
option_new          = 'New only'
option_all          = 'All'
option_shifted      = "Shifted forward"
option_hide         = '⊗ Hide'
icon_paint          = '🎨'

//////////////////////
// Functions
//////////////////////
f_if (_cond, _a, _b) => _cond ? _a : _b

f_pipfy (_v) => _v / (syminfo.mintick * 10)

f_pricefy (_p) => _p * (syminfo.mintick * 10)

f_grad (t, b, i) => color.from_gradient(i, 0, 100, t, b)

f_get_day (n) =>
    switch n
        1 => ['Sun', '日']
        2 => ['Mon', '月']
        3 => ['Tue', '火']
        4 => ['Wed', '水']
        5 => ['Thu', '木']
        6 => ['Fri', '金']
        7 => ['Sat', '土']

f_sub_days (_d, _s) =>
    _newDay = _d - _s
    _newDay > 7 ? 1 : _newDay == 0 ? 7 : _newDay

f_color (_name, _a, _b, _c, _d, _e, _f, _g, _h, _i, _j) =>
    switch _name
        'monokai'           => _a
        'monokaipro'        => _b
        'panda'             => _c
        'gruvbox'           => _d
        'spacemacs light'   => _e
        'spacemacs dark'    => _f
        'guardians'         => _g
        'mono'              => _h
        'tradingview'       => _i
        'custom'            => _j
        => _i

f_clear_lines (_arr, _min = 0) =>
    if array.size(_arr) > _min
        for i = 1 to array.size(_arr)
            line.delete(array.shift(_arr))

f_clear_labels (_arr, _min = 0) =>
    if array.size(_arr) > _min
        for i = 1 to array.size(_arr)
            label.delete(array.shift(_arr))

f_clear_boxes (_arr, _min = 0) =>
    if array.size(_arr) > _min
        for i = 1 to array.size(_arr)
            box.delete(array.shift(_arr))

f_pricerange_avg (_avg, _lookback) =>
    math.avg(_avg, _avg[_lookback], _avg[_lookback*2], _avg[_lookback*3])

f_candle (_x1, _y1, _x2, _y2, _border_color, _bgcolor, _width, _style, _xloc) =>
    box.new(_x1, _y1, _x2, _y2, _border_color, _width, _style, extend.none, _xloc, _bgcolor)

//  Code from "Volume Profile, Pivot Anchored by DGT" © dgtrd
f_drawOnlyBoxX(_left, _top, _right, _bottom, _border_color, _border_width, _border_style) =>
    box.new(_left, _top, _right, _bottom, _border_color, _border_width, _border_style, bgcolor=_border_color)

//  Code from "Volume Profile, Pivot Anchored by DGT" © dgtrd
f_getHighLow(_len, _calc) =>
    if _calc
        htf_l = low
        htf_h = high
        
        for x = 1 to _len
            htf_l := math.min(low [x], htf_l)
            htf_h := math.max(high[x], htf_h)

        [htf_h, htf_l]

//////////////////////
// Inputs
//////////////////////
i_htf_type                  = input.string('Auto', 'Time frame', options=['Auto', '15', '30', '60', '120', '240', '360', '720', 'D', 'W', 'M', '2M', '4M', '6M', '12M'], group=GP2)
i_color_name                = str.lower(input.string('monokai', icon_paint + ' ', options=['TradingView', 'monokai', 'monokaipro', 'panda', 'Gruvbox', 'Spacemacs light', 'Spacemacs dark', 'Guardians', 'mono', 'custom'], inline='color', group=GP2))
i_color_1                   = input.color(color.green   , '', inline='color', group=GP2)
i_color_2                   = input.color(color.red     , '', inline='color', group=GP2)
i_color_3                   = input.color(color.purple  , '', inline='color', group=GP2)
i_color_4                   = input.color(color.orange  , '', inline='color', group=GP2)
i_color_5                   = input.color(color.blue    , '', inline='color', group=GP2)

// HTF Candle
// =============
option_display1             = 'Both'
option_display2             = 'Open-Close'
option_display3             = 'High-Low'
i_candle_display            = input.string(option_display1, 'Display ', options=[option_display1, option_display2, option_display3, option_hide], inline='htf_display', group=GP3)
i_candle_show               = i_candle_display != option_hide
i_candle_thickness          = input.int(1, '', minval=0, inline='htf_display', group=GP3)
i_candle_line_transp        = 10
i_candle_bg_transp          = 96
i_candle_show_wick          = input.bool(true, "Wick", inline='htf_display', group=GP3) and i_candle_show
i_candle_show_wick         := i_candle_display == option_display3 ? false : i_candle_show_wick
i_candle_access             = input.string(option_shifted, "History", options=[option_all, option_new, option_shifted], group=GP3)
i_candle_show_hl            = i_candle_display == option_display3 or i_candle_display == option_display1
i_candle_show_oc            = i_candle_display == option_display2 or i_candle_display == option_display1

access_newonly              = i_candle_access == option_new
access_all                  = i_candle_access == option_all
access_shifted              = i_candle_access == option_shifted
shift                       = access_shifted ? 1 : 0

// Wick style
i_candle_wick_thickness     = i_candle_thickness
i_candle_wick_transp        = 3

// Time division
i_tdiv_number               = input.int(0, 'Divisions', options=[0, 2, 3, 4, 5, 6, 7, 8], group=GP3, inline='divisions_display', tooltip='Num. of divisions / Extend / Thickness')
i_tdiv_extend               = input.string('None', '', options=['None', 'Both'], group=GP3, inline='divisions_display')
i_tdiv_extend              := i_tdiv_extend == 'None' ? extend.none : extend.both
i_tdiv_thickness            = input.int(1, '', minval=0, maxval=10, group=GP3, inline='divisions_display')
i_tdiv_show                 = i_tdiv_number > 1
i_tdiv_style                = line.style_dotted

// Filled candle (negative only)
i_candle_show_fill          = input.bool(false, 'Filled negative candles', group=GP3)

// Labels
// =============
option_candle_label1        = 'Price'
option_candle_label2        = 'Pips'
option_sep1                 = 'dot'
option_sep2                 = 'br'
i_candle_label              = input.string(option_hide, 'Display ', options=[option_candle_label1, option_candle_label2, option_hide], inline='candle_labels', group=GP8)
i_candle_label_show         = i_candle_label != option_hide 
i_candle_label_size         = input.string(size.small, '', options=[size.auto, size.tiny, size.small, size.normal, size.large, size.huge], inline='candle_labels', group=GP8)
i_candle_label_sep          = input.string(option_sep1, '', options=[option_sep1, option_sep2], inline='candle_labels', group=GP8)
i_candle_label_position     = label.style_label_upper_left
i_candle_label_tz           = 'GMT+3'
i_labels_show_price         = i_candle_label == option_candle_label1
i_labels_show_pips          = i_candle_label == option_candle_label2
i_labels_show_htf           = input.bool(true , 'TF'            , inline='labels_options', group=GP8)
i_labels_show_avg           = input.bool(false, 'Avg'           , inline='labels_options', group=GP8)
i_labels_show_day           = input.bool(false, 'Day'           , inline='labels_options', group=GP8)
i_labels_show_tds           = input.bool(false, 'TD Seq.'       , inline='labels_options', group=GP8)
i_label_transp              = 10


// Pivots
// =============
option_pivot_label1         = 'Levels'
option_pivot_label2         = 'Levels & Price'
option_pivot_label3         = 'Price'
i_pivots_type               = input.string(option_hide, 'Display ', options=['Traditional', 'Fibonacci', 'Woodie', 'Classic', 'DM', 'Camarilla', 'Floor', 'Expected Pivot Points', option_hide], inline='pivots_display', group=GP1)
i_pivots_show               = i_pivots_type != option_hide
i_pivots_show_cpr           = input.bool(false, 'CPR', inline='pivots_display', group=GP1) and i_pivots_show
i_pivots_history            = input.string(option_new, 'History ', options=[option_all, option_new], inline='pivot_history', group=GP1)
i_pivots_show_forecast      = input.bool(false, 'Forecast', inline='pivot_history', group=GP1)
i_pivots_line_style         = input.string(line.style_dotted, 'Line   ', options=[line.style_solid, line.style_dotted, line.style_dashed], inline='pivot_line_style', group=GP1)
i_pivots_show_zone          = input.bool(true, 'BG', inline='pivot_line_style', group=GP1) and not (i_pivots_type == 'Traditional' or i_pivots_type == 'DM')
i_pivots_line_thickness     = 1
i_pivots_line_transp        = 30
i_pivots_label_position     = input.string(option_hide, 'Labels ', options=['Right', 'Left', option_hide], inline='pivot_label', group=GP1)
i_pivots_label_show         = option_hide != i_pivots_label_position
i_pivots_label              = input.string(option_pivot_label1, '', options=[option_pivot_label1, option_pivot_label3, option_pivot_label2], inline='pivot_label', group=GP1)
i_pivots_zone_transp        = 96
i_pivots_show_history       = i_pivots_history == option_all


// Volume Profile
// (Code from "Volume Profile, Pivot Anchored by DGT" © dgtrd) 🙏
// =====================
i_vp_history      = input.string(option_hide, 'History ', options=[option_all, option_new, option_hide], inline='BB3', group = GP5)
i_color_6         = input.color(color.new(color.orange, 35), ''                         , inline='BB3', group = GP5, tooltip="Color is enabled when GENERAL's Color is custom.")
i_color_7         = input.color(color.new(color.gray, 70), ''                           , inline='BB3', group = GP5)
isValueArea       = input.float(68, "Value Area Volume %", minval = 0, maxval = 100                   , group = GP5) / 100
i_show_poc        = input.bool(true, 'POC'                                              , inline='PoC', group = GP5)
i_color_8         = input.color(color.new(color.maroon, 20), ''                         , inline='PoC', group = GP5) // pocColor
profileWidth      = input.int(15, 'Profile Width %', minval = 0, maxval = 100                         , group = GP5) / 100
i_profileLevels     = input.int(25, 'Number of Rows' , minval = 10, maxval = 100 , step = 1             , group = GP5)
profilePlacement  = input.string('Left', 'Placment', options = ['Right', 'Left']                      , group = GP5)
i_show_vprofile   = i_vp_history != option_hide
i_poc_width       = 3

if access_newonly
    i_show_vprofile := false
    i_show_poc := false

// Expected price range
// =====================
option_epr3                 = 'Predict next'
option_epr4                 = 'Developing'
i_epr_history               = input.string(option_hide, 'Display', options=[option_all, option_new, option_epr3, option_epr4,  option_hide], group=GP6)
i_epr_show                  = i_epr_history !=  option_hide
i_epr_next                  = i_epr_history == option_epr3
i_epr_shifted               = i_epr_next
i_epr_2x                    = i_epr_next
i_epr_developing            = i_epr_history == option_epr4
i_epr_transp                = 97

// Options & Experimental
// ======================
i_clean_momory              = input.bool(false, 'Memory cleaning', group=GP9)
i_labels_day_kanji          = input.bool(false, 'Day (Kanji)', group=GP9)

i_drawing_max               = i_clean_momory ? 5 : 200

if i_clean_momory
    i_vp_history := option_new

//////////////////////
// Colors
//////////////////////
c_none  = color.new(color.black, 100)
c_bull  = f_color(i_color_name, ac.monokai('green'),    ac.monokaipro('green'),     ac.panda('green'),  ac.gruvbox('green'),    ac.spacemacs_light('green'),    ac.spacemacs_dark('green'),   #31a936,  ac.panda('light-gray'),   ac.tradingview_flag('green'),   i_color_1)
c_bear  = f_color(i_color_name, ac.monokai('pink'),     ac.monokaipro('pink'),      ac.panda('red'),    ac.gruvbox('red'),      ac.spacemacs_light('red'),      ac.spacemacs_dark('red'),     #75091e,  ac.panda('dark-gray'),    ac.tradingview_flag('red'),     i_color_2)
c_PP    = f_color(i_color_name, ac.monokai('purple'),   ac.monokaipro('purple'),    ac.panda('purple'), ac.gruvbox('purple'),   ac.spacemacs_light('magenta'),  ac.spacemacs_dark('magenta'), #a35fd1,  ac.panda('dark-gray'),    ac.tradingview_flag('purple'),  i_color_3)
c_RX    = f_color(i_color_name, ac.monokai('orange'),   ac.monokaipro('orange'),    ac.panda('orange'), ac.gruvbox('orange'),   ac.spacemacs_light('yellow'),   ac.spacemacs_dark('yellow'),  #dac23a,  ac.panda('gray'),         ac.tradingview_flag('orange'),  i_color_4)
c_SX    = c_RX
c_CPR   = c_PP
c_vpa   = f_color(i_color_name, color.new(ac.monokai('orange'), 40), color.new(ac.monokaipro('orange'), 40), color.new(ac.panda('orange'), 48), color.new(ac.gruvbox('yellow'), 38), color.new(ac.spacemacs_light('yellow'), 40), color.new(ac.spacemacs_dark('yellow'), 35), color.new(#dac23a, 35), color.new(ac.panda('darkgray'), 35), color.new(ac.tradingview_flag('orange'), 40), i_color_6)
c_vpd   = i_color_name != 'custom' ? color.new(f_grad(c_vpa, ac.panda('verylightgray'), 25), 75) : i_color_7
c_poc   = f_color(i_color_name, ac.monokai('pink'), ac.monokaipro('pink'), ac.panda('red'), ac.gruvbox('faded_red'), ac.spacemacs_light('red'), ac.spacemacs_dark('red'),  #75091e,  ac.panda('verydarkgray'), ac.tradingview_flag('pink'),  i_color_8)
c_poc  := i_color_name != 'custom' ? i_color_name == 'mono' ? ac.panda('verydarkgray') : color.new(f_grad(c_vpa, c_poc, 60), 10) : i_color_8
c_epr_upper = f_color(i_color_name, ac.monokai('blue'), ac.monokaipro('blue'), ac.panda('blue'), ac.gruvbox('blue'), ac.spacemacs_light('blue'), ac.spacemacs_dark('blue'), #2061c8, ac.panda('light-gray'), ac.tradingview_flag('blue'), i_color_5)
c_epr_lower = c_epr_upper
c_epr_base  = c_epr_upper

//////////////////////
// CALCULATIONS
//////////////////////
htf = i_htf_type == 'Auto' ? timeframe.isintraday ? timeframe.multiplier <= 60 ? 'D' : timeframe.multiplier <= 240 ? 'W' : 'M' : timeframe.period == 'D' ? '3M' : '12M' : i_htf_type
htf_changed = ta.change(time(htf))

[O1, H1, L1, C1, O0, H0, L0, C0] = f.htf_ohlc(htf)

////////////////////////
// RENDERING FUNCTIONS
////////////////////////
f_render_pivots_label (_x, _y, _text, _color, _style, _show) =>
    label id = na
    
    if _show
        v_price = str.format(f_price_shr, _y)
        v_text = ''

        if i_pivots_label == option_pivot_label1
            v_text := _text
        else if i_pivots_label == option_pivot_label2
            v_text := _text + ' (' + v_price + ')'
        else if i_pivots_label == option_pivot_label3
            v_text := v_price

        id := label.new(_x, _y, v_text, textcolor=_color, color=c_none, style=_style, size=size.normal)

    id
    
f_render_pivots_line (_x1, _y, _x2, _width, _color, _style, _show) =>
    id = (_show and _y > 0) ? line.new(_x1, _y, _x2, _y, width=_width, color=color.new(_color, i_pivots_line_transp), style=_style) : na

    id

f_render_pivots_box (_x1, _y1, _x2, _y2, _color, _show) =>
    id = _show ? box.new(_x1, _y1, _x2, _y2, bgcolor=color.new(_color, i_pivots_zone_transp), border_color=c_none) : na

    id

//label.new(bar_index, na, str.tostring(_start) + '\n' + str.tostring(_step) + '\n' + str.tostring(c), yloc=yloc.abovebar)

f_remove_lines (_arr, _max, _start, _step) =>
    if array.size(_arr) > (_max * _step)
        for i = _start to array.size(_arr) - 1
            line.delete(array.shift(_arr))

f_remove_linefills (_arr, _max, _start, _step) =>
    if array.size(_arr) > (_max * _step)
        for i = _start to array.size(_arr) - 1
            linefill.delete(array.shift(_arr))

f_remove_boxes (_arr, _max, _start, _step) =>
    if array.size(_arr) > (_max * _step)
        for i = _start to array.size(_arr) - 1
            box.delete(array.shift(_arr))

f_remove_labels (_arr, _max, _start, _step) =>
    if array.size(_arr) > (_max * _step)
        for i = _start to array.size(_arr) - 1
            label.delete(array.shift(_arr))

f_render_pivots (_show, _show_history, _next, _x1, _x2, _lines, _labels, _boxes, _should_delete) =>
    if _show
        O = _next ? O0 : O1
        H = _next ? H0 : H1
        L = _next ? L0 : L1
        C = _next ? C0 : C1

        [PP, R1, S1, R2, S2, R3, S3, R4, S4, R5, S5] = f.pivots(i_pivots_type, O, H, L, C)
        [BC, TC, CPR] = f.cpr(H, L, C)

        lines_start     = array.size(_lines)
        boxes_start     = array.size(_boxes)
        labels_start    = array.size(_labels)
        
        if (not _show_history) or _should_delete
            f_clear_lines(_lines, 0)
            f_clear_labels(_labels, 0)
            f_clear_boxes(_boxes, 0)

        // Lines
        array.push(_lines, f_render_pivots_line(_x1, PP, _x2, math.max(2, i_pivots_line_thickness), c_PP, line.style_solid, true) )
        array.push(_lines, f_render_pivots_line(_x1, R1, _x2, i_pivots_line_thickness, c_RX, i_pivots_line_style, true) )
        array.push(_lines, f_render_pivots_line(_x1, R2, _x2, i_pivots_line_thickness, c_RX, i_pivots_line_style, true) )
        array.push(_lines, f_render_pivots_line(_x1, R3, _x2, i_pivots_line_thickness, c_RX, i_pivots_line_style, true) )
        array.push(_lines, f_render_pivots_line(_x1, R4, _x2, i_pivots_line_thickness, c_RX, i_pivots_line_style, true) )
        array.push(_lines, f_render_pivots_line(_x1, R5, _x2, i_pivots_line_thickness, c_RX, i_pivots_line_style, true) )
        array.push(_lines, f_render_pivots_line(_x1, S1, _x2, i_pivots_line_thickness, c_SX, i_pivots_line_style, true) )
        array.push(_lines, f_render_pivots_line(_x1, S2, _x2, i_pivots_line_thickness, c_SX, i_pivots_line_style, true) )
        array.push(_lines, f_render_pivots_line(_x1, S3, _x2, i_pivots_line_thickness, c_SX, i_pivots_line_style, true) )
        array.push(_lines, f_render_pivots_line(_x1, S4, _x2, i_pivots_line_thickness, c_SX, i_pivots_line_style, true) )
        array.push(_lines, f_render_pivots_line(_x1, S5, _x2, i_pivots_line_thickness, c_SX, i_pivots_line_style, true) )
        array.push(_lines, f_render_pivots_line(_x1, BC, _x2, 1, c_CPR, line.style_dotted, i_pivots_show_cpr) )
        array.push(_lines, f_render_pivots_line(_x1, TC, _x2, 1, c_CPR, line.style_dotted, i_pivots_show_cpr) )

        // Boxes
        array.push(_boxes, f_render_pivots_box(_x1, R1, _x2, R2, c_RX , i_pivots_show_zone) )
        array.push(_boxes, f_render_pivots_box(_x1, R3, _x2, R4, c_RX , i_pivots_show_zone) )
        array.push(_boxes, f_render_pivots_box(_x1, S1, _x2, S2, c_SX , i_pivots_show_zone) )
        array.push(_boxes, f_render_pivots_box(_x1, S3, _x2, S4, c_SX , i_pivots_show_zone) )
        array.push(_boxes, f_render_pivots_box(_x1, BC, _x2, TC, c_CPR, i_pivots_show_zone and i_pivots_show_cpr) )

        // Labels
        label_x     = i_pivots_label_position == 'Left' ? _x1 : _x2
        label_style = i_pivots_label_position == 'Left' ? label.style_label_right : label.style_label_left
        array.push(_labels, f_render_pivots_label(label_x, PP, t_PP, color.new(c_PP , 20), label_style, i_pivots_label_show) )
        array.push(_labels, f_render_pivots_label(label_x, R5, t_R5, color.new(c_RX , 30), label_style, i_pivots_label_show) )
        array.push(_labels, f_render_pivots_label(label_x, R4, t_R4, color.new(c_RX , 30), label_style, i_pivots_label_show) )
        array.push(_labels, f_render_pivots_label(label_x, R3, t_R3, color.new(c_RX , 30), label_style, i_pivots_label_show) )
        array.push(_labels, f_render_pivots_label(label_x, R2, t_R2, color.new(c_RX , 30), label_style, i_pivots_label_show) )
        array.push(_labels, f_render_pivots_label(label_x, R1, t_R1, color.new(c_RX , 30), label_style, i_pivots_label_show) )
        array.push(_labels, f_render_pivots_label(label_x, S1, t_S1, color.new(c_SX , 30), label_style, i_pivots_label_show) )
        array.push(_labels, f_render_pivots_label(label_x, S2, t_S2, color.new(c_SX , 30), label_style, i_pivots_label_show) )
        array.push(_labels, f_render_pivots_label(label_x, S3, t_S3, color.new(c_SX , 30), label_style, i_pivots_label_show) )
        array.push(_labels, f_render_pivots_label(label_x, S4, t_S4, color.new(c_SX , 30), label_style, i_pivots_label_show) )
        array.push(_labels, f_render_pivots_label(label_x, S5, t_S5, color.new(c_SX , 30), label_style, i_pivots_label_show) )
        array.push(_labels, f_render_pivots_label(label_x, BC, t_BC, color.new(c_CPR, 30), label_style, i_pivots_label_show and i_pivots_show_cpr) )
        array.push(_labels, f_render_pivots_label(label_x, TC, t_TC, color.new(c_CPR, 30), label_style, i_pivots_label_show and i_pivots_show_cpr) )

        if _show_history and (not _should_delete)
            lines_step      = array.size(_lines)  - lines_start
            boxes_step      = array.size(_boxes)  - boxes_start
            labels_step     = array.size(_labels) - labels_start

            f_remove_lines(_lines  , i_drawing_max+1, lines_start , lines_step)
            f_remove_boxes(_boxes  , i_drawing_max+1, boxes_start , boxes_step)
            f_remove_labels(_labels, i_drawing_max+1, labels_start, labels_step)

///////////////////
// VolumeProfile //

f_render_vp (_x1, _x2, _px1, _px2, _nx1, _nx2, _show) =>
    barPriceLow         = low
    barPriceHigh        = high
    nzVolume            = nz(volume)

    var a_poc           = array.new<line>()
    var vp_box          = array.new<box>()
    var x1              = 0
    var x2              = 0
    var levelAbovePoc   = 0
    var levelBelowPoc   = 0
    tick                = syminfo.mintick * 50

    if _show
        if htf_changed
            x1 := access_shifted ? _x1 : _px1
            x2 := access_shifted ? _x2 : _px2

        profileLength = (x2 - x1) + 1

        [priceHighest, priceLowest] = f_getHighLow(profileLength, htf_changed)

        profileLevels = syminfo.type == 'forex' ?  math.round((priceHighest - priceLowest) / tick) : i_profileLevels

        priceStep = (priceHighest - priceLowest) / profileLevels
        
        volumeStorageT    = array.new_float(profileLevels + 1, 0.)

        if htf_changed and nzVolume and priceStep > 0 and bar_index > profileLength and profileLength > 0

            if i_vp_history == option_new
                f_clear_boxes(vp_box, 0)
                f_clear_lines(a_poc, 0)

            for barIndexx = 1 to profileLength
                level = 0
                barIndex = barIndexx

                for priceLevel = priceLowest to priceHighest by priceStep
                    if barPriceHigh[barIndex] >= priceLevel and barPriceLow[barIndex] < priceLevel + priceStep
                        array.set(volumeStorageT, level, array.get(volumeStorageT, level) + nzVolume[barIndex] * ((barPriceHigh[barIndex] - barPriceLow[barIndex]) == 0 ? 1 : priceStep / (barPriceHigh[barIndex] - barPriceLow[barIndex])) )
                    level += 1
        
            pocLevel          = array.indexof(volumeStorageT, array.max(volumeStorageT))
            totalVolumeTraded = array.sum(volumeStorageT) * isValueArea
            valueArea         = array.get(volumeStorageT, pocLevel)
            levelAbovePoc    := pocLevel
            levelBelowPoc    := pocLevel
            
            while valueArea < totalVolumeTraded
                if levelBelowPoc == 0 and levelAbovePoc == profileLevels - 1
                    break
        
                volumeAbovePoc = 0.
                if levelAbovePoc < profileLevels - 1 
                    volumeAbovePoc := array.get(volumeStorageT, levelAbovePoc + 1)
        
                volumeBelowPoc = 0.
                if levelBelowPoc > 0
                    volumeBelowPoc := array.get(volumeStorageT, levelBelowPoc - 1)
                
                if volumeBelowPoc == 0 and volumeAbovePoc == 0
                    break
                
                if volumeAbovePoc >= volumeBelowPoc
                    valueArea     += volumeAbovePoc
                    levelAbovePoc += 1
                else
                    valueArea     += volumeBelowPoc
                    levelBelowPoc -= 1
        
            for level = 0 to profileLevels - 1
                if i_show_vprofile
                    startBoxIndex   = profilePlacement == 'Right' ? x2 - int(array.get(volumeStorageT, level) / array.max(volumeStorageT) * profileLength * profileWidth) : x1
                    endBoxIndex     = profilePlacement == 'Right' ? x2 : x1 + int( array.get(volumeStorageT, level) / array.max(volumeStorageT) * profileLength * profileWidth)
                    boxY1           = priceLowest + ((level + 0.18) * priceStep)
                    boxY2           = priceLowest + ((level + 0.92) * priceStep)
                    array.push(vp_box, f_drawOnlyBoxX(startBoxIndex, boxY1, endBoxIndex, boxY2, level >= levelBelowPoc and level <= levelAbovePoc ? c_vpa : c_vpd, 1, line.style_solid) )

            if i_show_poc
                poc_y  = priceLowest + (array.indexof(volumeStorageT, array.max(volumeStorageT))) * priceStep
                array.push(a_poc, line.new(x1, poc_y, x2, poc_y, width=i_poc_width, color=c_poc))

///////////////
// HTF Candle
///////////////
f_render_tdiv (_x, _y1, _y2, _color, _transp, _show) =>
    id = _show ? line.new(_x, _y1, _x, _y2, color=color.new(_color, _transp), width=i_tdiv_thickness, style=i_tdiv_style, extend=i_tdiv_extend) : na
    id

f_get_div_x (_x1, _x2, _x3, _num) =>
    math.round((_x2 - _x1) * (1/i_tdiv_number*_num)) + _x3

// Divisions:
f_render_divs (x1, x2, px1, px2, nx1, nx2, color01, color11, wick_transp, _show) =>
    if _show
        var line split_1 = na, var line split_2 = na
        var line split_3 = na, var line split_4 = na
        var line split_5 = na, var line split_6 = na
        var line split_7 = na
        var line forecast_split_1 = na, var line forecast_split_2 = na
        var line forecast_split_3 = na, var line forecast_split_4 = na
        var line forecast_split_5 = na, var line forecast_split_6 = na
        var line forecast_split_7 = na
        var a_lines = array.new<line>()
        

        if htf_changed
            line.delete(split_1)
            line.delete(split_2)
            line.delete(split_3)
            line.delete(split_4)
            line.delete(split_5)
            line.delete(split_6)
            line.delete(split_7)
            
            if access_newonly or access_all or access_shifted
                split_x1 = f_get_div_x(x1, x2, x1, 1)
                split_x2 = f_get_div_x(x1, x2, x1, 2)
                split_x3 = f_get_div_x(x1, x2, x1, 3)
                split_x4 = f_get_div_x(x1, x2, x1, 4)
                split_x5 = f_get_div_x(x1, x2, x1, 5)
                split_x6 = f_get_div_x(x1, x2, x1, 6)
                split_x7 = f_get_div_x(x1, x2, x1, 7)
                split_1 := f_render_tdiv(split_x1, H0[shift], L0[shift], color01[shift], wick_transp, i_tdiv_number > 1)
                split_2 := f_render_tdiv(split_x2, H0[shift], L0[shift], color01[shift], wick_transp, i_tdiv_number > 2)
                split_3 := f_render_tdiv(split_x3, H0[shift], L0[shift], color01[shift], wick_transp, i_tdiv_number > 3)
                split_4 := f_render_tdiv(split_x4, H0[shift], L0[shift], color01[shift], wick_transp, i_tdiv_number > 4)
                split_5 := f_render_tdiv(split_x5, H0[shift], L0[shift], color01[shift], wick_transp, i_tdiv_number > 5)
                split_6 := f_render_tdiv(split_x6, H0[shift], L0[shift], color01[shift], wick_transp, i_tdiv_number > 6)
                split_7 := f_render_tdiv(split_x7, H0[shift], L0[shift], color01[shift], wick_transp, i_tdiv_number > 7)

            if access_all or access_shifted
                split_x1 = f_get_div_x(px1, px2, px1, 1)
                split_x2 = f_get_div_x(px1, px2, px1, 2)
                split_x3 = f_get_div_x(px1, px2, px1, 3)
                split_x4 = f_get_div_x(px1, px2, px1, 4)
                split_x5 = f_get_div_x(px1, px2, px1, 5)
                split_x6 = f_get_div_x(px1, px2, px1, 6)
                split_x7 = f_get_div_x(px1, px2, px1, 7)
                a_start  = array.size(a_lines)
                array.push(a_lines, f_render_tdiv(split_x1, H1[shift], L1[shift], color11[shift], wick_transp, i_tdiv_number > 1) )
                array.push(a_lines, f_render_tdiv(split_x2, H1[shift], L1[shift], color11[shift], wick_transp, i_tdiv_number > 2) )
                array.push(a_lines, f_render_tdiv(split_x3, H1[shift], L1[shift], color11[shift], wick_transp, i_tdiv_number > 3) )
                array.push(a_lines, f_render_tdiv(split_x4, H1[shift], L1[shift], color11[shift], wick_transp, i_tdiv_number > 4) )
                array.push(a_lines, f_render_tdiv(split_x5, H1[shift], L1[shift], color11[shift], wick_transp, i_tdiv_number > 5) )
                array.push(a_lines, f_render_tdiv(split_x6, H1[shift], L1[shift], color11[shift], wick_transp, i_tdiv_number > 6) )
                array.push(a_lines, f_render_tdiv(split_x7, H1[shift], L1[shift], color11[shift], wick_transp, i_tdiv_number > 7) )
                f_remove_lines(a_lines, i_drawing_max, a_start, array.size(a_lines) - a_start)

            if access_shifted
                split_x1 = math.min(maximum_x, f_get_div_x(x1, x2, nx1, 1))
                split_x2 = math.min(maximum_x, f_get_div_x(x1, x2, nx1, 2))
                split_x3 = math.min(maximum_x, f_get_div_x(x1, x2, nx1, 3))
                split_x4 = math.min(maximum_x, f_get_div_x(x1, x2, nx1, 4))
                split_x5 = math.min(maximum_x, f_get_div_x(x1, x2, nx1, 5))
                split_x6 = math.min(maximum_x, f_get_div_x(x1, x2, nx1, 6))
                split_x7 = math.min(maximum_x, f_get_div_x(x1, x2, nx1, 7))
                forecast_split_1 := f_render_tdiv(split_x1, H0, L0, color01, wick_transp, i_tdiv_number > 1), line.delete(forecast_split_1[1])
                forecast_split_2 := f_render_tdiv(split_x2, H0, L0, color01, wick_transp, i_tdiv_number > 2), line.delete(forecast_split_2[1])
                forecast_split_3 := f_render_tdiv(split_x3, H0, L0, color01, wick_transp, i_tdiv_number > 3), line.delete(forecast_split_3[1])
                forecast_split_4 := f_render_tdiv(split_x4, H0, L0, color01, wick_transp, i_tdiv_number > 4), line.delete(forecast_split_4[1])
                forecast_split_5 := f_render_tdiv(split_x5, H0, L0, color01, wick_transp, i_tdiv_number > 5), line.delete(forecast_split_5[1])
                forecast_split_6 := f_render_tdiv(split_x6, H0, L0, color01, wick_transp, i_tdiv_number > 6), line.delete(forecast_split_6[1])
                forecast_split_7 := f_render_tdiv(split_x7, H0, L0, color01, wick_transp, i_tdiv_number > 7), line.delete(forecast_split_7[1])

        else
            if access_newonly or access_all
                line.set_y1(split_1, H0), line.set_y2(split_1, L0)
                line.set_y1(split_2, H0), line.set_y2(split_2, L0)
                line.set_y1(split_3, H0), line.set_y2(split_3, L0)
                line.set_y1(split_4, H0), line.set_y2(split_4, L0)
                line.set_y1(split_5, H0), line.set_y2(split_5, L0)
                line.set_y1(split_6, H0), line.set_y2(split_6, L0)
                line.set_y1(split_7, H0), line.set_y2(split_7, L0)

            if access_shifted
                line.set_y1(forecast_split_1, H0), line.set_y2(forecast_split_1, L0), line.set_color(forecast_split_1, color.new(color01, wick_transp))
                line.set_y1(forecast_split_2, H0), line.set_y2(forecast_split_2, L0), line.set_color(forecast_split_2, color.new(color01, wick_transp))
                line.set_y1(forecast_split_3, H0), line.set_y2(forecast_split_3, L0), line.set_color(forecast_split_3, color.new(color01, wick_transp))
                line.set_y1(forecast_split_4, H0), line.set_y2(forecast_split_4, L0), line.set_color(forecast_split_4, color.new(color01, wick_transp))
                line.set_y1(forecast_split_5, H0), line.set_y2(forecast_split_5, L0), line.set_color(forecast_split_5, color.new(color01, wick_transp))
                line.set_y1(forecast_split_6, H0), line.set_y2(forecast_split_6, L0), line.set_color(forecast_split_6, color.new(color01, wick_transp))
                line.set_y1(forecast_split_7, H0), line.set_y2(forecast_split_7, L0), line.set_color(forecast_split_7, color.new(color01, wick_transp))
    true    

//////////////////////////
// Expected Price Range //

f_render_EPR (_x1, _x2, _px1, _px2, _nx1, _nx2, _avg, _show) =>
    var plot_ex1    = 0.0
    var plot_ex2    = 0.0
    var a_arr       = array.new<line>()
    var b_arr       = array.new<linefill>()

    scale           = i_epr_2x ? 1 : 0.5
    x1              = i_epr_shifted ? _nx1 : _x1
    x2              = i_epr_shifted ? _nx2 : _x2
    yc              = math.avg(H0, L0)
    y1              = yc + (_avg * scale)
    y2              = yc - (_avg * scale)
    plot_ex1       := y1
    plot_ex2       := y2

    if _show and (not i_epr_developing)
        var line epr_base          = na
        var line epr_upper         = na
        var line epr_lower         = na
        var linefill epr_upperfill = na
        var linefill epr_lowerfill = na

        if htf_changed
            if access_shifted
                epr_upper := line.new(x1, y1, x2, y1, width=2, color=c_epr_upper)
                epr_lower := line.new(x1, y2, x2, y2, width=2, color=c_epr_lower)
                epr_base  := line.new(x1, yc, x2, yc, width=1, color=c_epr_base, style=line.style_dotted)

                epr_upperfill := linefill.new(epr_base, epr_upper, color.new(c_epr_upper, i_epr_transp))
                epr_lowerfill := linefill.new(epr_base, epr_lower, color.new(c_epr_lower, i_epr_transp))
                
                a_start     = array.size(a_arr)
                b_start    = array.size(b_arr)
                array.push(a_arr, epr_upper)
                array.push(a_arr, epr_base)
                array.push(a_arr, epr_lower)
                array.push(b_arr, epr_upperfill)
                array.push(b_arr, epr_lowerfill)

                if i_epr_history == option_new or i_epr_history == option_epr3
                    line.delete(epr_upper[1])
                    line.delete(epr_lower[1])
                    linefill.delete(epr_upperfill[1])
                    linefill.delete(epr_lowerfill[1])
                    line.delete(epr_base[1])

                f_remove_lines    (a_arr, i_drawing_max + 1, a_start, array.size(a_arr) - a_start)
                f_remove_linefills(b_arr, i_drawing_max + 1, b_start, array.size(b_arr) - b_start)
        else
            if access_shifted
                line.set_y1(epr_upper, y1), line.set_y2(epr_upper, y1)
                line.set_x1(epr_upper, x1), line.set_x2(epr_upper, x2)

                line.set_y1(epr_lower, y2), line.set_y2(epr_lower, y2)
                line.set_x1(epr_lower, x1), line.set_x2(epr_lower, x2)

                line.set_y1(epr_base, yc), line.set_y2(epr_base, yc)
                line.set_x1(epr_base, x1), line.set_x2(epr_base, x2)

    [plot_ex1, plot_ex2]

////////////
// Labels //
f_label (_htf, _chg, _avg, _day, _td, _ts, _offset = 0) =>
    t = array.new<string>()
    
    //if _day != 0
    //    label.new(bar_index, na, str.tostring(day), yloc=yloc.abovebar, size=size.tiny, color=c_none, textcolor=color.gray)

    if i_labels_show_htf
        array.push(t, _htf)

    if i_labels_show_day
        [en, kanji] = f_get_day(_day[_offset])
        array.push(t, i_labels_day_kanji ? kanji : en) // array.push(t, f_get_day(day) + '('+ str.tostring(day) +')')

    if i_labels_show_price and i_labels_show_avg
        array.push(t, str.format(f_price, _chg) + str.format(' ('+ f_price +')', _avg))
    else if i_labels_show_price
        array.push(t, str.format(f_price, _chg))

    if i_labels_show_pips and i_labels_show_avg
        array.push(t, str.format(f_pip, f_pipfy(_chg)) + str.format(' ('+ f_pip_ +')', f_pipfy(_avg)))
    else if i_labels_show_pips
        array.push(t, str.format(f_pip, f_pipfy(_chg)))

    if i_labels_show_tds and (_td > 0 or _ts > 0)
        t_td = _td > 0 ? '▲'+str.tostring(_td) : ''
        t_ts = _ts > 0 ? '▼'+str.tostring(_ts) : ''
    
        array.push(t, t_td + t_ts)
    
    array.join(t, i_candle_label_sep == option_sep1 ? separator_dot : separator_br)

f_render_labels (_htf, x1, x2, px1, px2, nx1, nx2, color01, color11, _avg, _day, _td0, _ts0, _td1, _ts1, _show) =>
    if _show
        var label note      = na
        var label n_note    = na
        var a_labels        = array.new<label>()

        p_offset            = 0
        c_offset            = 0

        if access_all
            p_offset := (x1 - px1) * 1  // 1 day  ago
        else if access_shifted
            p_offset := (x1 - px1) * 2  // 2 days ago
            c_offset := (x1 - px1) * 1  // 1 day  ago

        v_note_n    = f_label(_htf, H0[shift] - L0[shift], _avg, _day, _td1, _ts1)
        v_note_c    = f_label(_htf, H1[shift] - L1[shift], _avg, _day, _td0[shift], _ts0[shift], c_offset)
        v_note_p    = f_label(_htf, H1[shift] - L1[shift], _avg, _day, _td0[shift], _ts0[shift], p_offset)

        if htf_changed
            label.delete(note)
            label.delete(n_note)

            if access_newonly or access_all or access_shifted
                note := label.new(x1, L0[shift], v_note_c, style=i_candle_label_position, color=c_none, textcolor=color.new(color01[shift], i_label_transp), size=i_candle_label_size, textalign=text.align_left)

            if access_all or access_shifted
                a_start = array.size(a_labels)
                array.push(a_labels, label.new(px1, L1[shift], v_note_p, style=i_candle_label_position, color=c_none, textcolor=color.new(color11[shift], i_label_transp), size=i_candle_label_size, textalign=text.align_left))
                f_remove_labels(a_labels, i_drawing_max, a_start, 1)

            if access_shifted
                n_note := label.new(nx1, L0, v_note_n, style=i_candle_label_position, color=c_none, textcolor=color.new(color01, i_label_transp), size=i_candle_label_size, textalign=text.align_left)
        else
            if access_newonly or access_all
                label.set_y(note, L0)
                label.set_text(note, v_note_n)
                label.set_textcolor(note, color.new(color01, i_label_transp))

            else if access_shifted
                label.set_text(note, v_note_c)
                
                label.set_y(n_note, L0)
                label.set_text(n_note, v_note_n)
                label.set_textcolor(n_note, color.new(color01, i_label_transp))

        true
    true


//////////
// Wick //
f_render_wick (_x1, _x2, _xc, _px1, _px2, _pxc, _nx1, _nx2, _nxc, color01, color11, _show) =>
    if _show
        var line h = na
        var line l = na
        //var line p_h = na
        //var line p_l = na
        var line n_h = na
        var line n_l = na
        var a_lines = array.new<line>()

        if htf_changed
            line.delete(h)
            line.delete(l)
            line.delete(n_h)
            line.delete(n_l)

            if access_newonly or access_all or access_shifted
                x = _xc
                h := line.new(x, H0[shift], x, math.max(O0[shift], C0[shift]), color=color.new(color01[shift], i_candle_wick_transp), width=i_candle_wick_thickness)
                l := line.new(x, L0[shift], x, math.min(O0[shift], C0[shift]), color=color.new(color01[shift], i_candle_wick_transp), width=i_candle_wick_thickness)

            if access_all or access_shifted
                a_start = array.size(a_lines)
                x = _pxc
                array.push(a_lines, line.new(x, H1[shift], x, math.max(O1[shift], C1[shift]), color=color.new(color11[shift], i_candle_wick_transp), width=i_candle_wick_thickness) )
                array.push(a_lines, line.new(x, L1[shift], x, math.min(O1[shift], C1[shift]), color=color.new(color11[shift], i_candle_wick_transp), width=i_candle_wick_thickness) )
                f_remove_lines(a_lines, i_drawing_max, a_start, array.size(a_lines) - a_start)

            if access_shifted
                x = _nxc
                n_h := line.new(x, H0, x, math.max(O0, C0), color=color.new(color01, i_candle_wick_transp), width=i_candle_wick_thickness), line.delete(n_l[1])
                n_l := line.new(x, L0, x, math.min(O0, C0), color=color.new(color01, i_candle_wick_transp), width=i_candle_wick_thickness), line.delete(n_h[1])
                
        else
            if access_newonly or access_all
                line.set_y1(h, H0)
                line.set_y2(h, math.max(O0, C0))
                line.set_color(h, color.new(color01, i_candle_wick_transp))
                
                line.set_y1(l, L0)
                line.set_y2(l, math.min(O0, C0))
                line.set_color(l, color.new(color01, i_candle_wick_transp))

            if access_shifted
                x = _nxc - math.round(i_candle_wick_thickness / 2)
                line.set_x1(n_h, x), line.set_x2(n_h, x)
                line.set_y1(n_h, H0)
                line.set_y2(n_h, math.max(O0, C0))
                line.set_color(n_h, color.new(color01, i_candle_wick_transp))
                
                line.set_x1(n_l, x), line.set_x2(n_l, x)
                line.set_y1(n_l, L0)
                line.set_y2(n_l, math.min(O0, C0))
                line.set_color(n_l, color.new(color01, i_candle_wick_transp))
        true
    true

/////////////
// CANDLES //

f_render_candles (_x1, _x2, _px1, _px2, _nx1, _nx2, color0, color01, color1, color11, _show) =>
    if _show
        var box hl      = na
        var box oc      = na
        var box p_hl    = na
        var box p_oc    = na
        var box n_oc    = na
        var box n_hl    = na
        var a = array.new<box>()

        i_hl_bg_transp      = (i_candle_thickness == 0 and i_candle_show_hl) ? 90  : 96
        i_oc0_bg_transp     = (i_candle_thickness == 0 and i_candle_show_hl) ? 100 : (i_candle_thickness == 0) ? 90 : 94
        i_oc1_bg_transp     = (i_candle_thickness == 0 and i_candle_show_hl) ? 100 : (i_candle_thickness == 0) ? 90 : 94
        _width = i_candle_thickness

        if i_candle_show_fill
            i_oc0_bg_transp := O0[shift] > C0[shift] ? 0 : i_oc0_bg_transp
            i_oc1_bg_transp := O1[shift] > C1[shift] ? 0 : i_oc1_bg_transp

        if htf_changed
            box.delete(hl)
            box.delete(oc)
            box.delete(n_oc)
            box.delete(n_hl)

            if access_newonly or access_all or access_shifted
                if i_candle_show_hl
                    hl := f_candle(_x1, H0[shift], _x2, L0[shift], color01[shift], color.new(color0[shift], i_hl_bg_transp), _width, line.style_dotted, xloc.bar_index)
                if i_candle_show_oc
                    oc := f_candle(_x1, O0[shift], _x2, C0[shift], color01[shift], color.new(color0[shift], i_oc0_bg_transp), _width, line.style_solid, xloc.bar_index)

            if access_all or access_shifted
                a_start = array.size(a)

                if i_candle_show_hl
                    array.push(a, f_candle(_px1, H1[shift], _px2, L1[shift], color11[shift], color.new(color1[shift], i_hl_bg_transp), _width, line.style_dotted, xloc.bar_index))
                if i_candle_show_oc
                    array.push(a, f_candle(_px1, O1[shift], _px2, C1[shift], color11[shift], color.new(color1[shift], i_oc1_bg_transp), _width, line.style_solid, xloc.bar_index))
                
                f_remove_boxes(a, i_drawing_max, a_start, array.size(a) - a_start)

            if access_shifted
                if i_candle_show_hl
                    n_hl := f_candle(_nx1, H0, _nx2, L0, color01, color0, _width, line.style_dotted, xloc.bar_index), box.delete(n_hl[1])
                if i_candle_show_oc
                    n_oc := f_candle(_nx1, O0, _nx2, C0, color01, color0, _width, line.style_solid, xloc.bar_index), box.delete(n_oc[1])

        else
            if access_newonly or access_all
                box.set_top(hl, H0)
                box.set_bottom(hl, L0)
                box.set_bgcolor(hl, color.new(color0, i_hl_bg_transp))
                box.set_border_color(hl, color01)
            
                box.set_top(oc, math.max(O0, C0))
                box.set_bottom(oc, math.min(O0, C0))
                box.set_bgcolor(oc, color.new(color0, i_oc0_bg_transp))
                box.set_border_color(oc, color01)

            if access_shifted
                box.set_left(n_oc, _nx1)
                box.set_right(n_oc, _nx2)
                box.set_top(n_oc, math.max(O0, C0))
                box.set_bottom(n_oc, math.min(O0, C0))
                box.set_bgcolor(n_oc, color.new(color0, i_oc0_bg_transp))
                box.set_border_color(n_oc, color01)

                box.set_left(n_hl, _nx1)
                box.set_right(n_hl, _nx2)
                box.set_top(n_hl, H0)
                box.set_bottom(n_hl, L0)
                box.set_bgcolor(n_hl, color.new(color0, i_hl_bg_transp))
                box.set_border_color(n_hl, color01)

//////////
// MAIN //

f_render_main (_htf, _trans, _btransp) =>
    // for exports variables
    var plot_o = 0.0, var plot_o0 = 0.0
    var plot_c = 0.0, var plot_c0 = 0.0
    var plot_h = 0.0, var plot_h0 = 0.0
    var plot_l = 0.0, var plot_l0 = 0.0
    var price_range     = 0.0
    var day             = 0
    var TD0             = 0
    var TS0             = 0
    var pvt_lines       = array.new<line>(), var pvt_labels         = array.new<label>(), var pvt_boxes         = array.new<box>()
    var pvt_lines_next  = array.new<line>(), var pvt_labels_next    = array.new<label>(), var pvt_boxes_next    = array.new<box>()
    var pvt_lines_past  = array.new<line>(), var pvt_labels_past    = array.new<label>(), var pvt_boxes_past    = array.new<box>()

    color0  = O0 < C0 ? color.new(c_bull, _trans)   : color.new(c_bear, _trans)
    color01 = O0 < C0 ? color.new(c_bull, _btransp) : color.new(c_bear, _btransp)
    color1  = O1 < C1 ? color.new(c_bull, _trans)   : color.new(c_bear, _trans)
    color11 = O1 < C1 ? color.new(c_bull, _btransp) : color.new(c_bear, _btransp)

    // Positions
    // 
    // |----P----| |--------|  |----N----|
    // px1     px2 x1      x2 nx1       nx2
    //     pxc         xc          nxc

    px1     = ta.valuewhen(htf_changed, bar_index, 1)
    x1      = ta.valuewhen(htf_changed, bar_index, 0)
    px2     = x1 - 1
    pxc     = math.round(math.avg(px2, px1))
    x2      = (px2 - px1) + x1
    xc      = math.round(math.avg(x2, x1))
    nx1     = (2 * x1) - px1
    nx2     = math.min((x2 - x1) + nx1, bar_index + 500)
    nxc     = math.round(math.avg(nx2, nx1))
    
    four_days_ago = (x1 - px1) * 4

    price_range_avg = f_pricerange_avg(price_range, x2 - x1)

    //a = label.new(px1, close, 'px1', yloc=yloc.price), label.delete(a[1])
    //b = label.new( x1, close,  'x1', yloc=yloc.price), label.delete(b[1])
    //d = label.new( x2, close,  'x2', yloc=yloc.price), label.delete(d[1])
    //c = label.new(nx1, close, 'nx1', yloc=yloc.price , style=label.style_label_up), label.delete(c[1])
    //e = label.new(nx2, close, 'nx2', yloc=yloc.price), label.delete(e[1])

    if htf_changed
        price_range := (H1 - L1)
        day         := dayofweek(time, i_candle_label_tz)
        TD0         := C0 > C0[four_days_ago] ? TD0 + 1 : 0
        TS0         := C0 < C0[four_days_ago] ? TS0 + 1 : 0

        if access_newonly or access_all or access_shifted
            plot_o := O0[shift], plot_c := C0[shift]
            plot_h := H0[shift], plot_l := L0[shift]
            plot_o0 := O0, plot_c0 := C0
            plot_h0 := H0, plot_l0 := L0
            f_render_pivots(i_pivots_show, i_pivots_show_history, false, x1 , x2 , pvt_lines, pvt_labels, pvt_boxes, false)

    else
        if access_newonly or access_all
            plot_o := O0, plot_c := C0
            plot_h := H0, plot_l := L0

        if access_shifted
            f_render_pivots(i_pivots_show_forecast, false, true, nx1, nx2, pvt_lines_next, pvt_labels_next, pvt_boxes_next, true)

    TD1 = C0 > C0[four_days_ago] ? TD0 + 1 : 0
    TS1 = C0 < C0[four_days_ago] ? TS0 + 1 : 0

    [plot_ex1, plot_ex2] = f_render_EPR(x1, x2, px1, px2, nx1, nx2, price_range_avg, i_epr_show)
    f_render_wick(x1, x2, xc, px1, px2, pxc, nx1, nx2, nxc, color01, color11, i_candle_show_wick)
    f_render_divs(x1, x2, px1, px2, nx1, nx2, color01, color11, i_candle_wick_transp, i_tdiv_show)
    f_render_candles(x1, x2, px1, px2, nx1, nx2, color0, color01, color1, color11, i_candle_show)
    f_render_vp(x1, x2, px1, px2, nx1, nx2, i_show_vprofile)
    f_render_labels(_htf, x1, x2, px1, px2, nx1, nx2, color01, color11, price_range_avg, day, TD0, TS0, TD1, TS1, i_candle_label_show)
    
    //label.new(bar_index, na, str.tostring(array.size(pvt_lines_next)), yloc=yloc.abovebar)

    [plot_o, plot_h, plot_l, plot_c, plot_o0, plot_h0, plot_l0, plot_c0, plot_ex1, plot_ex2]

[o, h, l, c, o0, h0, l0, c0, ex1, ex2] = f_render_main(htf, i_candle_bg_transp, i_candle_line_transp)


///////////////
// Plotting
///////////////
plot(o, 'HTF Open'  , linewidth=2, display=display.none)
plot(h, 'HTF High'  , linewidth=2, display=display.none)
plot(l, 'HTF Low'   , linewidth=2, display=display.none)
plot(c, 'HTF Close' , linewidth=2, display=display.none)
plot(i_epr_show and i_epr_developing ? ex1 : na, 'EPR top'    , linewidth=2, color=c_epr_upper)
plot(i_epr_show and i_epr_developing ? ex2 : na, 'EPR bottom' , linewidth=2, color=c_epr_lower)
plotshape((not htf_changed) and ta.cross(o, close) ? o : na, 'Crossed open' , color=color.orange, style=shape.xcross, location=location.absolute, size=size.tiny, display=display.none)
plotshape((not htf_changed) and ta.cross(h, close) ? h : na, 'Crossed high' , color=color.orange, style=shape.xcross, location=location.absolute, size=size.tiny, display=display.none)
plotshape((not htf_changed) and ta.cross(l, close) ? l : na, 'Crossed low'  , color=color.orange, style=shape.xcross, location=location.absolute, size=size.tiny, display=display.none)
plotshape((not htf_changed) and ta.cross(c, close) ? c : na, 'Crossed close', color=color.orange, style=shape.xcross, location=location.absolute, size=size.tiny, display=display.none)
plotshape((not htf_changed) and (i_candle_access == option_new or i_candle_access == option_all) and ta.change(h) > 0, 'New high', style=shape.labelup, color=color.blue, location=location.bottom, size=size.tiny, display=display.none)  // Work on 'new only' or 'All'
plotshape((not htf_changed) and (i_candle_access == option_new or i_candle_access == option_all) and ta.change(l) < 0, 'New low', style=shape.labeldown, color=color.red, location=location.bottom, size=size.tiny, display=display.none)   // Work on 'new only' or 'All'

