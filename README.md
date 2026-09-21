# iPhone 2 Android 4 Messages

Transfer your SMS messages &amp; MMS attachments from iPhone to Android! Unfortunately, the Android Switch app doesn't reliably copy text messages unless several poorly documented conditions are met (iMessages off, sync cable vs. wifi, etc.). By using an unencrypted iTunes backup, this script should allow you to transfer your messages via XML files accepted by [SMS Backup & Restore](https://www.synctech.com.au/sms-backup-restore/).

The converter runs locally. It opens the Apple backup read-only and does not upload messages, images, phone numbers, or templates.

## Important limitations

- This is an independent community project, not an Apple, Google, or SyncTech product.
- Test with a pilot before performing a full restore. Back up the Android phone first.
- iMessage and RCS history is represented as Android SMS/MMS history.
- Group-chat structure, replies, edits, unsends, delivery state, effects, and reactions cannot be reproduced exactly.
- Text is restored separately from images. Each image becomes an image-only MMS at its original timestamp.
- Multiple images from one Apple message become separate MMS bubbles a millisecond apart.
- Images are converted to high-quality JPEG display copies for compatibility. Source files are never modified.
- Animated images use their first frame. Video, audio, documents, and other non-image attachments are reported but not converted.
- Conversation matching depends on phone numbers/email identifiers in the Apple database and may be imperfect.

## Requirements

- Windows 10 or later
- Python 3.10 or later
- An unencrypted iPhone/iPad backup made with Apple Devices or iTunes
- SMS Backup & Restore on the destination Android phone
- Enough free disk space for Base64-encoded XML and JPEG working files

## Installation

1. Extract the release archive.
2. Run `setup.bat`.
3. Open Command Prompt in the extracted directory.

`setup.bat` creates a local `.venv` and installs the pinned dependencies from `requirements.txt`.

## Choose a cutoff

If the Android phone has already received messages, use an exclusive cutoff corresponding to the completed cutover. This prevents the Apple backup from overlapping newer Android history.

The cutoff must include a UTC offset or `Z`. Example:

```text
2026-09-08T06:00:00Z
```

Omit `--cutoff` only when every message in the Apple backup should be converted.

## Find the Apple backup

When exactly one backup exists in the standard Windows location, `--backup` may be omitted. If multiple backups exist, pass the desired backup directory explicitly:

```bat
--backup "C:\Users\YOUR_NAME\AppData\Roaming\Apple Computer\MobileSync\Backup\BACKUP_ID"
```

## Workflow

The examples below use `run.bat`, which activates the project virtual environment automatically.

### 1. Preflight

```bat
run.bat preflight --backup "BACKUP_DIRECTORY" --cutoff "CUTOFF"
```

Review `report-preflight.json`. Confirm that the message/image counts and disk estimate are plausible.

### 2. Text pilot

```bat
run.bat pilot-text --backup "BACKUP_DIRECTORY" --cutoff "CUTOFF"
```

Restore the generated `iphone-text-pilot-*.xml` files. Check incoming/outgoing direction, timestamps, and several conversations. Leave successful pilot records on the phone.

### 3. Full text conversion

```bat
run.bat convert-text --backup "BACKUP_DIRECTORY" --cutoff "CUTOFF"
```

The converter reads `pilot-manifest.json` and omits pilot records. Restore the generated `iphone-text-full-*.xml` files.

### 4. Create native image-only templates

SMS Backup & Restore relies on device/provider-specific MMS metadata. Create two known-good templates on the destination Android phone:

1. Disable RCS temporarily.
2. Receive one MMS containing exactly one image and no caption/text.
3. Send one MMS containing exactly one image and no caption/text.
4. Back up those records with SMS Backup & Restore.
5. Keep the resulting incoming and outgoing XML files private; they contain phone numbers and image data.

You may supply two separate XML files or XML files containing additional records. The converter selects the first suitable incoming and outgoing MMS records.

### 5. Image pilot

```bat
run.bat pilot-images --backup "BACKUP_DIRECTORY" --cutoff "CUTOFF" --incoming-template "INCOMING.xml" --outgoing-template "OUTGOING.xml"
```

Restore `iphone-images-pilot-*.xml`. Confirm that all four images appear in the correct conversations with plausible timestamps. Leave successful pilot images on the phone.

### 6. Full image conversion

```bat
run.bat convert-images --backup "BACKUP_DIRECTORY" --cutoff "CUTOFF" --incoming-template "INCOMING.xml" --outgoing-template "OUTGOING.xml"
```

The converter omits images recorded in `pilot-manifest.json` and writes year-split `iphone-images-full-*.xml` files. Review `report-convert-images.json` before restoring them.

## Output directory

The default is:

```text
Desktop\iPhoneMessagesToAndroid
```

Override it with `--output "DIRECTORY"`. The output may contain private message metadata and image data; protect it accordingly.

## Starting over or intentional duplicates

The full commands normally require `pilot-manifest.json` so successful pilot records are not duplicated. Use `--include-pilot` only when the pilot records were deleted or were never restored.

## Troubleshooting

- **Every MMS is invalid:** Confirm both native templates are image-only MMS records without captions. RCS records are not suitable.
- **Images are missing from the conversation list:** They are inserted at their original timestamps. Open the corresponding older conversation.
- **An image conversion fails:** Check `report-convert-images.json`. Unsupported or damaged files are skipped rather than fabricated.
- **`sms.db` is missing:** Re-create the Apple backup with local backup encryption disabled.
- **Duplicate messages:** Do not restore the same pilot/full XML twice. Retain `pilot-manifest.json` between stages.

## Privacy and security

The tool does not need network access after dependency installation. Apple backups, native Android template XMLs, generated XMLs, reports, and JPEG caches can all contain sensitive personal information. Do not publish them as bug-report fixtures without sanitizing them.

## License

Project code and documentation are released under the MIT License. Dependencies retain their own licenses; see [THIRD_PARTY_NOTICES.md](THIRD_PARTY_NOTICES.md).
