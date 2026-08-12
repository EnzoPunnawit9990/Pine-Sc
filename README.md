// This source code is subject to the terms of the Mozilla Public License 2.0 at 

//@version=5
indicator("Summary Dashboard Only", shorttitle="Dashboard", overlay = true)

// ==========================================
// 1. Settings (การตั้งค่าที่จำเป็นสำหรับคำนวณ Dashboard)
// ==========================================
len1 = input.int(20, title="EMA 1 Length", group="Indicator Settings")
len2 = input.int(50, title="EMA 2 Length", group="Indicator Settings")
len3 = input.int(200, title="EMA 3 Length", group="Indicator Settings")

col1 = input.color(color.new(#3aa6ff, 0), title="EMA 1 Color", group="Indicator Settings")
col2 = input.color(color.new(#ffab2d, 0), title="EMA 2 Color", group="Indicator Settings")
col3 = input.color(color.new(#ff0000, 0), title="EMA 3 Color", group="Indicator Settings")

vwapSource = input.source(hlc3, title="VWAP Source", group="Indicator Settings")
vwapColor  = input.color(color.new(#8400ff, 0), title="VWAP Color", group="Indicator Settings")

summaryLoc  = input.string("Top Right", "Dashboard Location", options=["Top Right", "Top Center", "Top Left", "Bottom Right", "Bottom Left"], group="Dashboard Settings")

// ==========================================
// 2. Calculations (คำนวณค่าเพื่อนำไปแสดงผล)
// ==========================================
// คำนวณ EMA และ VWAP 
ema1 = ta.ema(close, len1)
ema2 = ta.ema(close, len2)
ema3 = ta.ema(close, len3)
vwapValue = ta.vwap(vwapSource)

// ฟังก์ชั่นคำนวณแนวโน้ม (1 = Uptrend, -1 = Downtrend, 0 = Sideway)
f_trend_calc(l1, l2, l3) =>
    e1 = ta.ema(close, l1)
    e2 = ta.ema(close, l2)
    e3 = ta.ema(close, l3)
    e1 > e2 and e2 > e3 ? 1 : (e1 < e2 and e2 < e3 ? -1 : 0)

// ดึงข้อมูล Trend ของแต่ละ Timeframe
trend_1m  = request.security(syminfo.tickerid, "1", f_trend_calc(len1, len2, len3))
trend_15m = request.security(syminfo.tickerid, "15", f_trend_calc(len1, len2, len3))
trend_30m = request.security(syminfo.tickerid, "30", f_trend_calc(len1, len2, len3))
trend_45m = request.security(syminfo.tickerid, "45", f_trend_calc(len1, len2, len3))
trend_1h  = request.security(syminfo.tickerid, "60", f_trend_calc(len1, len2, len3))
trend_1d  = request.security(syminfo.tickerid, "D", f_trend_calc(len1, len2, len3))

// ดึงข้อมูลราคาเปิด, สูงสุด, ต่ำสุด ของวันนี้
[d_open, d_high, d_low] = request.security(syminfo.tickerid, "D", [open, high, low])

// ==========================================
// 3. Dashboard UI (สร้างและแสดงผลตาราง)
// ==========================================
var sum_pos = summaryLoc == "Top Right" ? position.top_right :
              summaryLoc == "Top Center" ? position.top_center :
              summaryLoc == "Top Left" ? position.top_left :
              summaryLoc == "Bottom Right" ? position.bottom_right : position.bottom_left

// สร้างตารางขนาด 2 คอลัมน์ x 16 แถว
var sum_tb = table.new(sum_pos, 2, 16, bgcolor=color.new(#000207, 10), border_color=#000207, border_width=1, frame_color=#000207, frame_width=1)

// ฟังก์ชันย่อยสำหรับวาดข้อความ Trend ลงในตาราง
f_render_trend(tb, row, tf_name, trend_val) =>
    txt = "Sideway"
    col = color.gray
    if trend_val == 1
        txt := "Uptrend"
        col := color.green
    else if trend_val == -1
        txt := "Downtrend"
        col := color.red
    table.cell(tb, 0, row, tf_name, text_color=color.white, text_size=size.small)
    table.cell(tb, 1, row, txt, text_color=col, text_size=size.small)

if barstate.islast
    // Header หลัก
    table.cell(sum_tb, 0, 0, "Item / Timeframe", text_color=color.white, bgcolor=color.new(#000207, 0), text_size=size.small)
    table.cell(sum_tb, 1, 0, "Status / Value", text_color=color.white, bgcolor=color.new(#000207, 0), text_size=size.small)
    
    // --- สถานะอินดิเคเตอร์ปัจจุบัน ---
    table.cell(sum_tb, 0, 1, "EMA " + str.tostring(len1), text_color=col1, text_size=size.small)
    table.cell(sum_tb, 1, 1, close > ema1 ? "Price Above" : "Price Below", text_color=close > ema1 ? color.green : color.red, text_size=size.small)
    
    table.cell(sum_tb, 0, 2, "EMA " + str.tostring(len2), text_color=col2, text_size=size.small)
    table.cell(sum_tb, 1, 2, close > ema2 ? "Price Above" : "Price Below", text_color=close > ema2 ? color.green : color.red, text_size=size.small)
    
    table.cell(sum_tb, 0, 3, "EMA " + str.tostring(len3), text_color=col3, text_size=size.small)
    table.cell(sum_tb, 1, 3, close > ema3 ? "Price Above" : "Price Below", text_color=close > ema3 ? color.green : color.red, text_size=size.small)
    
    table.cell(sum_tb, 0, 4, "VWAP", text_color=vwapColor, text_size=size.small)
    table.cell(sum_tb, 1, 4, close > vwapValue ? "Price Above" : "Price Below", text_color=close > vwapValue ? color.green : color.red, text_size=size.small)
    
    // --- ราคาประจำวัน (Today Range) ---
    table.cell(sum_tb, 0, 5, "--- Today Range ---", text_color=color.yellow, bgcolor=color.new(#000207, 0), text_size=size.small)
    table.cell(sum_tb, 1, 5, "", bgcolor=color.new(#000207, 0))

    table.cell(sum_tb, 0, 6, "Today Open", text_color=color.white, text_size=size.small)
    table.cell(sum_tb, 1, 6, str.tostring(d_open, format.mintick), text_color=color.white, text_size=size.small)

    table.cell(sum_tb, 0, 7, "Today High", text_color=color.white, text_size=size.small)
    table.cell(sum_tb, 1, 7, str.tostring(d_high, format.mintick), text_color=color.green, text_size=size.small)

    table.cell(sum_tb, 0, 8, "Today Low", text_color=color.white, text_size=size.small)
    table.cell(sum_tb, 1, 8, str.tostring(d_low, format.mintick), text_color=color.red, text_size=size.small)

    // --- แนวโน้มราย Timeframe (Multi-TF Trend) ---
    table.cell(sum_tb, 0, 9, "--- Trend Analysis ---", text_color=color.yellow, bgcolor=color.new(#000207, 0), text_size=size.small)
    table.cell(sum_tb, 1, 9, "", bgcolor=color.new(#000207, 0))

    f_render_trend(sum_tb, 10, "1M Trend", trend_1m)
    f_render_trend(sum_tb, 11, "15M Trend", trend_15m)
    f_render_trend(sum_tb, 12, "30M Trend", trend_30m)
    f_render_trend(sum_tb, 13, "45M Trend", trend_45m)
    f_render_trend(sum_tb, 14, "1H Trend", trend_1h)
    f_render_trend(sum_tb, 15, "1D Trend", trend_1d)
