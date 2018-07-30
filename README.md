This wraps Skia's C interface up in a way that allows it to be pulled in as a dependency for a Swift Package Manager project.

Note that the compiled `libskia.a` is not committed in this project, as it is enourmous - you have to build it yourself, and tell SPM where to find it when you link. 

# Usage

The client project should include this repo in its dependencies, like so:

```swift
dependencies: [
    .package(url: "git@github.com:elegantchaos/CSkia", from: "1.0.0"),
],
```

The client can then use the C interface like so:

```swift
import CSkia
import Foundation

func sk_color_set_argb(_ a: UInt32, _ r: UInt32, _ g: UInt32, _ b: UInt32) -> sk_color_t {
    return (((a) << 24) | ((r) << 16) | ((g) << 8) | (b))
}

func make_surface(_ width: Int32, _ height: Int32) -> OpaquePointer {
    var info = sk_imageinfo_t()
    info.width = width
    info.height = height
    info.colorType = sk_colortype_get_default_8888()
    return sk_surface_new_raster(&info, nil)
}

func draw(_ canvas: OpaquePointer) {
    let fill = sk_paint_new()
    sk_paint_set_color(fill, sk_color_set_argb(0xFF, 0x00, 0x00, 0xFF))
    sk_canvas_draw_paint(canvas, fill)

    sk_paint_set_color(fill, sk_color_set_argb(0xFF, 0x00, 0xFF, 0xFF))
    var rect = sk_rect_t()
    rect.left = 100.0
    rect.top = 100.0
    rect.right = 540.0
    rect.bottom = 380.0
    sk_canvas_draw_rect(canvas, &rect, fill)

    let stroke = sk_paint_new();
    sk_paint_set_color(stroke, sk_color_set_argb(0xFF, 0xFF, 0x00, 0x00))
    sk_paint_set_antialias(stroke, true)
    sk_paint_set_stroke(stroke, true)
    sk_paint_set_stroke_width(stroke, 5.0)
    let path = sk_path_new();

    sk_path_move_to(path, 50.0, 50.0)
    sk_path_line_to(path, 590.0, 50.0)
    sk_path_cubic_to(path, -490.0, 50.0, 1130.0, 430.0, 50.0, 430.0)
    sk_path_line_to(path, 590.0, 430.0)
    sk_canvas_draw_path(canvas, path, stroke)

    sk_paint_set_color(fill, sk_color_set_argb(0x80, 0x00, 0xFF, 0x00))
    var rect2 = sk_rect_t()
    rect2.left = 120.0
    rect2.top = 120.0
    rect2.right = 520.0
    rect2.bottom = 360.0
    sk_canvas_draw_oval(canvas, &rect2, fill)

    sk_path_delete(path);
    sk_paint_delete(stroke);
    sk_paint_delete(fill);
}

func emit_png(_ path: String, _ surface: OpaquePointer) {
    let image = sk_surface_new_image_snapshot(surface)
    if let encoded = sk_image_encode(image) {
        let raw = sk_data_get_data(encoded)
        let size = sk_data_get_size(encoded)
        let data = Data(bytes: raw!, count:size)
        try? data.write(to: URL(fileURLWithPath: path))
        sk_data_unref(encoded)
    }
    sk_image_unref(image)
}

let surface = make_surface(640, 480)
if let canvas = sk_surface_get_canvas(surface) {
    draw(canvas)
    emit_png("skia-c-example.png", surface)
    sk_surface_unref(surface)
}
```
