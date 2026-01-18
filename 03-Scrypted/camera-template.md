# Camera Configuration Template

## Camera Information

- **Camera Name**: 
- **Manufacturer**: 
- **Model**: 
- **Firmware Version**: 
- **Location**: 

## Network Configuration

- **IP Address**: 
- **MAC Address**: 
- **VLAN**: Camera Network
- **Port**: 
- **Protocol**: RTSP/HTTP/ONVIF

## Stream URLs

### Main Stream
- **URL**: `rtsp://IP_ADDRESS:PORT/stream1`
- **Resolution**: 
- **FPS**: 
- **Codec**: 
- **Bitrate**: 

### Sub Stream
- **URL**: `rtsp://IP_ADDRESS:PORT/stream2`
- **Resolution**: 
- **FPS**: 
- **Codec**: 
- **Bitrate**: 

## Scrypted Configuration

### Plugin Used
- **Plugin Name**: 
- **Plugin Version**: 

### Settings

```json
{
  "name": "",
  "ip": "",
  "username": "admin",
  "password": "${CAMERA_PASSWORD}"
}
```

### Transcoding Settings
- **Codec**: 
- **Resolution**: 
- **Bitrate**: 
- **Hardware Acceleration**: Yes/No

## HomeKit Secure Video

- **Enabled**: Yes/No
- **Recording**: 
- **Activity Zones**: 
- **Streaming Quality**: 

## Motion Detection

### Detection Settings
- **Type**: Camera Native / Scrypted / AI
- **Sensitivity**: 
- **Cooldown Period**: 
- **Recording Pre-buffer**: 
- **Recording Post-buffer**: 

### Detection Zones

Document zones using coordinates or description:
1. **Zone Name**: Description
2. 

## Recording Configuration

### Recording Schedule
- [ ] Continuous (24/7)
- [ ] Motion-activated only
- [ ] Scheduled times: 

### Storage
- **Location**: 
- **Retention Period**: 
- **Storage Quota**: 
- **Cleanup Policy**: 

## AI Features

- **Object Detection**: Enabled/Disabled
- **Detected Objects**: Person, Vehicle, Animal, etc.
- **Face Recognition**: Enabled/Disabled
- **License Plate Recognition**: Enabled/Disabled

## Notifications

### Notification Rules
- **Platform**: Home Assistant / iOS / Android
- **Trigger**: Motion / Person / Vehicle
- **Include Snapshot**: Yes/No
- **Rich Notifications**: Yes/No

### Notification Configuration

```yaml
# Home Assistant notification automation
```

## Camera Settings

### Image Settings
- **Brightness**: 
- **Contrast**: 
- **Saturation**: 
- **Sharpness**: 
- **WDR/HDR**: 

### Night Vision
- **IR Mode**: Auto/On/Off
- **IR Range**: 
- **Color Night Vision**: Yes/No

### Audio
- **Audio Enabled**: Yes/No
- **Microphone**: Yes/No
- **Speaker**: Yes/No
- **2-Way Audio**: Yes/No

## Power

- **Power Type**: PoE / 12V / Battery
- **Power Consumption**: 
- **UPS Backup**: Yes/No

## Mounting

- **Mount Type**: 
- **Field of View**: 
- **Height**: 
- **Angle**: 

## Maintenance

- **Last Cleaned**: 
- **Last Firmware Update**: 
- **Maintenance Schedule**: 
- **Known Issues**: 

## Troubleshooting

### Common Issues
1. **Issue**: 
   - **Solution**: 

## Performance

- **Average Bitrate**: 
- **CPU Usage**: 
- **Storage per Day**: 
- **Network Bandwidth**: 

## Related Documentation

- 
