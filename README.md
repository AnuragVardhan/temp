//In component.css
.gm-scrollbar-container {
  position: relative;
  overflow: hidden!important;
  width: 100%;
  height: 100%;
}
.gm-scrollbar-container .gm-scroll-view {
  width: 100%;
  height: 100%;
  overflow: scroll;
  -webkit-overflow-scrolling: touch;
}

/* @option: autoshow */
.gm-scrollbar-container.gm-autoshow .gm-scrollbar {
  opacity: 0;
  transition: opacity 120ms ease-out;
}
.gm-scrollbar-container.gm-autoshow:hover .gm-scrollbar,
.gm-scrollbar-container.gm-autoshow:focus .gm-scrollbar {
  opacity: 1;
  transition: opacity 340ms ease-out;
}
.gm-prevented .gm-scrollbar {
  display: none;
}
.gm-scrollbar {
  position: absolute;
  right: 2px;
  bottom: 2px;
  z-index: 1;
  border-radius: 3px;
}

.gm-scrollbar.-vertical {
  width: 6px;
  top: 2px;
}

.gm-scrollbar.-horizontal {
  height: 6px;
  left: 2px;
}

.gm-scrollbar .thumb {
  position: relative;
  width: 0;
  height: 0;
  cursor: pointer;
  border-radius: inherit;
  background-color: rgba(0,0,0,.2);
}

.gm-scrollbar .thumb:hover,
.gm-scrollbar .thumb:active {
  background-color: rgba(0,0,0,.3);
}

.gm-scrollbar.-vertical .thumb {
  width: 100%;
}

.gm-scrollbar.-horizontal .thumb {
  height: 100%;
}

.gm-scrollbar-disable-selection {
  -webkit-touch-callout: none;
  -webkit-user-select: none;
  -khtml-user-select: none;
  -moz-user-select: none;
  -ms-user-select: none;
  user-select: none;
}

.gm-prevented {
  -webkit-overflow-scrolling: touch;
}
