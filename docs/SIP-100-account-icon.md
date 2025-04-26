# SIP-100: Account Icon Display Using Metadata

## Summary

This proposal introduces a new feature that enables Symbol Explorer to render account icons based on image URLs stored in account metadata.  
By adding the owner's social media URL that references their address, it becomes possible to objectively verify that the address belongs to the owner.

## Motivation

Currently, accounts in Symbol Explorer are displayed only as raw address strings.  
By associating accounts with visual icons, the following benefits can be achieved:

- A more intuitive and user-friendly interface
- Promotion of decentralized identity and personalization
- Easier visual recognition of familiar addresses
- Verification of address ownership using social media links

## Specification

### Metadata Usage

- Metadata key: `social_meta_data`
- Value: JSON format

```json
{
  "url": "https://example.com",
  "name": "myname",
  "imageUrl": "https://example.com/image.jpg",
  "namespace": "my namespace"
}
```

### Display Rules

- If `social_meta_data` exists and contains a valid `imageUrl`, display the image as the account icon.
- If no valid metadata is found, fall back to the default SVG placeholder icon.
- The icon image should be displayed as a circle (`border-radius: 50%`).

### Moderation

- Users can manually hide icons they find inappropriate by clicking a "Hide" button.
- Hidden accounts are stored in the browser's localStorage to prevent future loading.
- (Optional) A "Report" button may be introduced to allow users to report inappropriate images for moderation.

### Validation

Before displaying an image, perform the following validations:

- Check if the URL starts with `http://` or `https://`
- Ensure the URL is within a reasonable length and proper format to prevent abuse

## Rationale

This feature leverages the existing Symbol account metadata functionality and does not require any protocol-level changes.  
It improves the user interface while maintaining backward compatibility and ensuring local client-side control for user safety.

## Backwards Compatibility

This proposal does not break any existing functionality.  
Accounts without `social_meta_data` will continue to display the default SVG icon as before.

## Implementation Notes

- Fetch metadata asynchronously to avoid blocking the initial page load
- Optimize icon size (e.g., within 64x64px)
- Consider implementing lazy loading for account images to enhance performance

## References

- Related feature discussion: [GitHub Issue #1200](https://github.com/symbol/symbol-explorer/issues/1200)

---

*Author: [bootarou]*  
*Date: 2025-04-26*
