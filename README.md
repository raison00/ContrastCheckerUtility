
A Contrast Checker utility and a matching UI debugger component for Android 

## In Android, WCAG 2.1 standards require a contrast ratio of at least 4.5:1 for normal text and 3.0:1 for large text/icons.
Accessibility-First Design.  Ensure color pairs are readable by calculating the Luminance Ratio at runtime.

- Formula: $(L_{brightest} + 0.05) / (L_{darkest} + 0.05)$Target: $\ge 4.5:1$ for standard content. 
- Implementation: Success extension utilizes this to verify that onSuccess remains legible even when the system injects dynamic user colors.

## Material 3 Color Scheme Reference
This table outlines the standard M3 color roles. Use these accessors to ensure your UI automatically adapts to both Light/Dark Mode and Dynamic User Themes.

- Contrast Ratios: When using .tertiary or .error, ensure you are using the matching .on... color for text. Material 3 guarantees these pairs meet WCAG contrast standards.

- Container vs. Base: Use .primary for small, high-impact items (buttons) and .primaryContainer for large blocks of color (cards) to avoid visual fatigue in Dark Mode.

- Dynamic Theming: Remember that these values are not static hex codes; they change based on the user's wallpaper if Dynamic Color is enabled on Android 12+.

| Element Type             | API Accessor | Context / Usage | Accessibility Pair |
| :---------------- | ------: | ----: | ----: |
| Primary Action       |   .primary   | Main FABs, primary buttons, active states. | .onPrimary |
| Primary Container |   .primaryContainer  | Less prominent than Primary; good for card headers. | .onPrimaryContainer |
| Soft Accent    |  .secondaryContainer  | Toggle states, chips, or subtle highlights. | .onSecondaryContainer |
| Success / Balance |  .tertiary   | Used for "Success" states, badges, or "New" alerts. | .onTertiary |
| Main Surface     |   .surface  | The main "paper" background of the app. | .onSurface |
| Input Decoration  |   .surfaceVariant  | Search bar backgrounds or inactive text field fills. | .onSurfaceVariant |
| Borders / Lines    |  .outline   | High-contrast borders for accessibility. | N/A |
| Subtle Dividers |  .outlineVariant  | Low-contrast separators between list items. | N/A |
| Danger / Alert    |  .error   | Validation errors or destructive "Delete" buttons. | .onError |
| Error Feedback |  .errorContainer   | Backgrounds for error banners or invalid input fields. | .onErrorContainer |

## Custom Semantic Roles in Material 3
Material 3 provides a robust error role but omits a dedicated success role to remain flexible for dynamic theming. Adding a Semantic Extension pattern to ensure consistency:

- Avoids "Magic Hex Codes": Instead of hardcoding Color.Green throughout the app, use MaterialTheme.colorScheme.success.

- Automatic Dark Mode: The extension detects the current brightness and swaps between a deep forest green (Light Mode) and a vibrant mint green (Dark Mode) in this example.

- Accessibility First: By defining onSuccess alongside success, we guarantee that text and icons always meet WCAG contrast requirements regardless of the user's theme.

## Usage Example
``` kotlin
Icon(
    imageVector = Icons.Default.CheckCircle,
    tint = MaterialTheme.colorScheme.success // Automatically picks the right green
)
```

# Accessibility Check Composable
``` kotlin
@Composable
fun AccessibilityCheck() {
    val bgColor = MaterialTheme.colorScheme.tertiary
    val textColor = MaterialTheme.colorScheme.onTertiary

    // Directly use the extension function
    val ratio = textColor.contrastAgainst(bgColor)

    Text(
        text = "Contrast Ratio: ${"%.2f".format(ratio)}",
        color = if (ratio >= 4.5f) Color.White else Color.Yellow
    )
}
```


# Accessible ContrastCheckerUtility
``` kotlin
import androidx.compose.ui.graphics.Color
import androidx.compose.ui.graphics.luminance
import kotlin.math.max
import kotlin.math.min

object ContrastChecker {
    /**
     * Calculates the contrast ratio between two colors.
     * Formula: (L1 + 0.05) / (L2 + 0.05)
     */
    fun calculateContrast(foreground: Color, background: Color): Float {
        val lum1 = foreground.luminance()
        val lum2 = background.luminance()
        val brightest = max(lum1, lum2)
        val darkest = min(lum1, lum2)
        return (brightest + 0.05f) / (darkest + 0.05f)
    }

    fun isAccessible(foreground: Color, background: Color): Boolean {
        return calculateContrast(foreground, background) >= 4.5f
    }
}
``` 
## Accessibility Debugger UI Component
``` kotlin
@Composable
fun ThemeAccessibilityDashboard() {
    val scheme = MaterialTheme.colorScheme
    val pairs = listOf(
        "Primary" to (scheme.primary to scheme.onPrimary),
        "Tertiary (Success)" to (scheme.tertiary to scheme.onTertiary),
        "Error" to (scheme.errorContainer to scheme.onErrorContainer)
    )

    Column(modifier = Modifier.padding(16.dp)) {
        Text("Theme Accessibility Check", style = MaterialTheme.typography.headlineSmall)
        
        pairs.forEach { (label, colors) ->
            val ratio = ContrastChecker.calculateContrast(colors.first, colors.second)
            val pass = ratio >= 4.5f

            Row(
                modifier = Modifier
                    .fillMaxWidth()
                    .padding(vertical = 8.dp)
                    .background(colors.second) // Background
                    .padding(8.dp),
                horizontalArrangement = Arrangement.SpaceBetween
            ) {
                Text(label, color = colors.first) // Foreground
                Text(
                    text = if (pass) "PASS (${"%.2f".format(ratio)})" else "FAIL",
                    color = if (pass) Color(0xFF4CAF50) else Color(0xFFF44336),
                    fontWeight = FontWeight.Bold
                )
            }
        }
    }
}
```
