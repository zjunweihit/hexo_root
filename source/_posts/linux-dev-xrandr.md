---
title: xrandr
date: 2016-07-03 14:34:55
categories:
  - linux-dev
  - Graphics
tags:
  - Xorg
---

X Resize and Rotate, is a extension for X server. xrandr is a tool of x11-xserver-utils.

<!--more-->
# How does xrandr get modes #
* `modes` indicates the resolution and refreshing for a monitor.
* xinit or startx to start X server
```
$ xrandr
or
$ xrandr -q
Screen 0: minimum 320 x 200, current 1920 x 1200, maximum 16384 x 16384
VGA-0 connected 1920x1200+0+0 (normal left inverted right x axis y axis) 518mm x 324mm
   1920x1200      60.0*+
   1600x1200      60.0
   1280x1024      75.0     60.0
   1152x864       75.0
   1024x768       75.1     60.0
   800x600        75.0     60.3
   640x480        75.0     60.0
   720x400        70.1
DisplayPort-0 disconnected (normal left inverted right x axis y axis)
HDMI-A-0 disconnected (normal left inverted right x axis y axis)
```
* The basic process of getting modes
```
xrandr -> X server(xrandr extension) -> libdrm -> kernel(DRM) -> get EDID from the monitor
```
* Code path
```
--------------------------------------------------
xrandr

  main() [xrandr.c]
  {
    /* to show mode name, such as 1920x1200 */
    jmode = find_mode_by_xid (output_info->modes[j]);
    printf (" ");
    printf ("  %-12s", jmode->name);
    ...

    /* If the name is same, continue
     * There is 12 modes in the mode list, skipping the mode with same name
     **/
    for (k = j; k < output_info->nmode; k++)
    {
        ...
        if (strcmp (jmode->name, kmode->name) != 0) continue;
        ...
    }
  }
  -> get_screen()
     -> XRRGetScreenResourcesCurrent()
        or
        XRRGetScreenResources()

--------------------------------------------------
X server

  get the command from ProcRandrVector[] [randr/rrdispatch.c]
  ProcRRGetScreenResources() [randr/rrscreen.c]
    rrGetScreenResources()
      RRGetInfo() [randr/rrinfo.c]
        *pScrPriv->rrGetInfo()
        It's xf86RandR12GetInfo12() [hw/xfree86/modes/xf86RandR12.c]
          xf86ProbeOutputModes() [hw/xfree86/modes/xf86Crtc.c]
            output_modes = (*output->funcs->get_modes) (output); [hw/xfree86/modes/xf86Crtc.c]

        ddx
        ------------------------------------------
            .get_modes = drmmode_output_get_modes in ddx
              {
              mode is converted from kmode via drmmode_ConvertFromKMode(, &koutput->modes[i], )
              drmModeConnectorPtr koutput = drmmode_output->mode_output;
              drmmode_output_private_ptr drmmode_output = output->driver_private;

              drmmode_output->mode_output = koutput; in drmmode_output_init()
              output->driver_private = drmmode_output
              koutput = drmModeGetConnector();
              }

--------------------------------------------------
libdrm

  drmModeGetConnector() [xf86drmMode.c]
    _drmModeGetConnector()
      drmIoctl(fd, DRM_IOCTL_MODE_GETCONNECTOR, &conn)

--------------------------------------------------
kernel(DRM)

  drm_mode_getconnector() [drivers/gpu/drm/drm_crtc.c]
    connector->funcs->fill_modes()
    it's drm_helper_probe_single_connector_modes() [drivers/gpu/drm/drm_probe_helper.c]
      drm_helper_probe_single_connector_modes_merge_bits()
        drm_load_edid_firmware(connector);
        count = (*connector_funcs->get_modes)(connector);
        it's amdgpu_connector_dp_get_modes() [drivers/gpu/drm/amd/amdgpu/amdgpu_connectors.c]
          amdgpu_connector_encoder_get_dp_bridge_encoder_id() to setup ddc on the bridge
          amdgpu_connector_get_edid()
          amdgpu_connector_ddc_get_modes()
            drm_add_edid_modes() add modes from EDID data [drivers/gpu/drm/drm_edid.c]
              add_detailed_modes()
              add_cvt_modes()
              ...
```
