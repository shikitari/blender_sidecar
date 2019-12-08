# Optimization of Blender for sidecar ( macOS + iPad )

![screen capture](/images/logo.gif)

## summry

- To attempt to optimize Blender for sidecar (macOS + iPad).
- I bought macbook so that using blender. but ...
- [Youtube](https://youtu.be/9nO9vem3Smw)

## Testing Environment

- macbook pro 2019
- iPad pro 2018
- Blender 2.8x

## WARRING

- It is makeshift solution.
- NO WARRANTY.
- WIP

## How to build blender from sorce oode

[link](https://wiki.blender.org/wiki/Building_Blender/Mac)

## My Optimazation

### 1. Stroke-in and Stroke-out is high(full) pressure. Sculpt & Paint mode

- modified source code.
- diff is bellow
- [developer.blender.org](https://developer.blender.org/T62565)

```C
--- a/source/blender/editors/sculpt_paint/paint_stroke.c
+++ b/source/blender/editors/sculpt_paint/paint_stroke.c
@@ -37,6 +37,7 @@

 #include "RNA_access.h"

+//#include "BKE_global.h"
 #include "BKE_context.h"
 #include "BKE_paint.h"
 #include "BKE_brush.h"
@@ -1335,6 +1336,8 @@ int paint_stroke_modal(bContext *C, wmOperator *op, const wmEvent *event)
   bool first_modal = false;
   bool redraw = false;
   float pressure;
+  static bool is_tablet_prev_event = false;
+  static uint seqNo = 0;

   if (event->type == INBETWEEN_MOUSEMOVE && !paint_tool_require_inbetween_mouse_events(br, mode)) {
     return OPERATOR_RUNNING_MODAL;
@@ -1345,6 +1348,56 @@ int paint_stroke_modal(bContext *C, wmOperator *op, const wmEvent *event)
                   1.0f :
                   WM_event_tablet_data(event, &stroke->pen_flip, NULL));

+//  if (G.debug & G_DEBUG_EVENTS) {
+//    printf("  ---\n");
+//    printf("  event->type:%d / event->val:%d \n", event->type, event->val);
+//    printf("  event->prevtype:%d / event->prevval:%d \n", event->prevtype, event->prevval);
+//    printf("  pressure:%ff / last_pressure:%ff \n", pressure, stroke->last_pressure);
+//    printf("  br->flag:%d \n", br->flag);
+//    printf("  is_tablet_current_event:%d \n",  (event->tablet_data) ? true : false);
+//    printf("  is_tablet_prev_event:%d \n", is_tablet_prev_event);
+//    printf("  seqNo:%d \n", seqNo);
+//  }
+
+  // If both the brush-mode is space and the brush-pressure is on.
+  if (br->flag & (BRUSH_SPACE) && br->flag & (BRUSH_ALPHA_PRESSURE | BRUSH_SIZE_PRESSURE)) {
+    // Sometime first press-event of pencil will be very high pressure so I will weaken pressure.
+    if (event->type == LEFTMOUSE && event->prevval == 1 && event->tablet_data) {
+      if (pressure >= 0.01f) {
+        pressure = 0.01f;
+      }
+    }
+
+    if ((event->type == LEFTMOUSE && event->val == KM_PRESS) || (event->type == LEFTMOUSE && event->val == KM_RELEASE)) {
+      seqNo = 0;
+    } else if (event->type == MOUSEMOVE && event->val == KM_PRESS) {
+       seqNo++;
+    }
+  }
+
+  // Dspite using only a pencil (no touching a screen, no clicking), a mouse-release-event without a tablet_data is occured.
+  // It's a just a guess, but If a pen is releasing from screen (is hovering), can't fetch pressure. Presumably It's a hardware constraint.
+  // So I will weaken pressure of mouse-left-release-event after mouse-event with tablet_data.
+  if (event->type == LEFTMOUSE && event->val == KM_RELEASE && is_tablet_prev_event) {
+    pressure = 0.0f;
+  }
+
+  // Somehow, last mouse-move-event hasn't tablet_data so ignore it.
+  // Moreover, pressure is 1.0f so I will weken it.
+  if (event->type == MOUSEMOVE && event->val == KM_PRESS && is_tablet_prev_event && !(event->tablet_data)) {
+    pressure = 0.0f;
+    is_tablet_prev_event = true;
+  } else {
+    // save previous is-tablet-flag.
+    is_tablet_prev_event = (event->tablet_data) ? true : false;
+  }
+  
+  if (seqNo <= 1) {
+    pressure = 0.0f;
+  }
+
+//  printf("  adjusted pressure:%ff \n", pressure);
+
   paint_stroke_add_sample(p, stroke, event->mval[0], event->mval[1], pressure);
   paint_stroke_sample_average(stroke, &sample_average);

```

### 2. Adjust pen pressure more easily. Can't  be set pressure in iPad

![screen capture](/images/capture.gif)

- develop addon that adjust pen-pressure easily
- IN PREPARATION

### Increase pinch-in(Zooming) sensibility

```C
--- a/intern/ghost/intern/GHOST_SystemCocoa.mm
+++ b/intern/ghost/intern/GHOST_SystemCocoa.mm
@@ -1729,7 +1729,7 @@ - (void)windowWillClose:(NSNotification *)notification
                                         GHOST_kTrackpadEventMagnify,
                                         x,
                                         y,
-                                        [event magnification] * 125.0 + 0.1,
+                                        [event magnification] * 500.0 + 0.1,
                                         0));
     } break;
 
```

### 4. Using s-Key can't sample color, due to no hovering in iPad

```C
--- a/source/blender/editors/sculpt_paint/paint_image.c
+++ b/source/blender/editors/sculpt_paint/paint_image.c
@@ -1057,7 +1057,7 @@ static int sample_color_modal(bContext *C, wmOperator *op, const wmEvent *event)

   switch (event->type) {
     case MOUSEMOVE: {
-      ARegion *ar = CTX_wm_region(C);
+          ARegion *ar = CTX_wm_region(C);
       RNA_int_set_array(op->ptr, "location", event->mval);
       paint_sample_color(C, ar, event->mval[0], event->mval[1], use_sample_texture, false);
       WM_event_add_notifier(C, NC_BRUSH | NA_EDITED, brush);
@@ -1068,11 +1068,11 @@ static int sample_color_modal(bContext *C, wmOperator *op, const wmEvent *event)
       if (event->val == KM_PRESS) {
         ARegion *ar = CTX_wm_region(C);
         RNA_int_set_array(op->ptr, "location", event->mval);
-        paint_sample_color(C, ar, event->mval[0], event->mval[1], use_sample_texture, true);
-        if (!data->sample_palette) {
-          data->sample_palette = true;
-          sample_color_update_header(data, C);
-        }
+        paint_sample_color(C, ar, event->mval[0], event->mval[1], use_sample_texture, false);
+//        if (!data->sample_palette) {
+//          data->sample_palette = true;
+//          sample_color_update_header(data, C);
+//        }
         WM_event_add_notifier(C, NC_BRUSH | NA_EDITED, brush);
       }
       break;
```
