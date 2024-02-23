---
title: "Spfx React Hook Tooltip Mouseposition"
date: 2024-02-21T17:08:27Z
draft: true
---


function StatusBox({ status, appName, details }: { status: string, appName: string, details: string }): JSX.Element {
  // ... existing code ...

  const [mousePosition, setMousePosition] = useState<{x: number, y: number} | null>(null);

  const handleMouseEnter = (event: React.MouseEvent<HTMLElement>) => {
    setMousePosition({ x: event.clientX, y: event.clientY });
    toggleIsCalloutVisible();
  };

  // ... existing code ...

  return (
    <div style={{ backgroundColor, color, padding: '10px', borderRadius: '5px', display: 'flex', alignItems: 'center' }}>
      <h1 style={{ marginRight: '10px' }}>{appName}</h1>
      <h3 id={detailsId} onMouseEnter={handleMouseEnter} onMouseLeave={toggleIsCalloutVisible}>{details}</h3>
      {isCalloutVisible && (
        <Callout
          ariaLabelledBy={labelId}
          ariaDescribedBy={descriptionId}
          role="dialog"
          className={styles2.callout}
          target={mousePosition ? { left: mousePosition.x, top: mousePosition.y, width: 0, height: 0 } : undefined}
          isBeakVisible={isBeakVisible}
          beakWidth={beakWidth}
          onDismiss={toggleIsCalloutVisible}
          directionalHint={directionalHint}
          setInitialFocus
        >
          <Text block variant="xLarge" className={styles2.title} id={labelId}>
            {appName}
          </Text>
          <Text block variant="small" id={descriptionId}>
            {details}
          </Text>
          <Link href="http://microsoft.com" target="_blank" className={styles2.link}>
            Sample link
          </Link>
        </Callout>
      )}
    </div>
  );
}
```

In this code, the handleMouseEnter function captures the mouse coordinates when the onMouseEnter event is triggered and sets them as the state. This state is then used as the target for the Callout component. This will make the callout appear at the position of the mouse when the onMouseEnter event is triggered.