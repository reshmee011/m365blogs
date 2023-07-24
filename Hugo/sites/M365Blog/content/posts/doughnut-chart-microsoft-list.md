---
title: "Expanding Doughnut Chart in Microsoft Lists: Visualize Data in Segments and Track Progress"
date: 2023-07-23T14:49:19+01:00
tags: ["Microsoft", "List","Doughnut Chart"]
draft: false
---

# Expanding Doughnut Chart in Microsoft Lists: Visualize Data in Segments and Track Progress

The doughnut chart is a powerful visualization tool that allows users to represent data in segments and track progress against targets. Frederico Sapia's [Doughnut Chart with percentage and values displayed](https://github.com/pnp/List-Formatting/blob/master/view-samples/custom-charts/Doughnut-Chart-PV.json) is a valuable custom chart for Microsoft Lists, enabling users to visualize data effectively. However, the original chart allowed for only six slices, limiting its flexibility. In this blog post, we explore how to extend the doughnut chart to support up to nine slices, and we'll also demonstrate how to display the percentage of each segment within the chart. A potential use scenerio is showing progress of how many have filled a company wide survey against what's remaining. 

The final doughnut chart with 10 slices

![Doughnut PV gif](../images/doughnut-chart/DoughnutChart.gif)

## Steps to extend the doughnut chart

To extend it to 9 slices, I followed these steps

1. Create the additional columns, COLORx, LABELx and VALUEx for the 3 additional slices. You can download the Excel from [doughnut chart](../images/doughnut-chart/Doughnut-Chart_v1.xlsx) to export to Sharepoint following instructions from [Export an Excel table to SharePoint](https://support.microsoft.com/en-us/office/export-an-excel-table-to-sharepoint-974544f9-94bc-4aa8-9159-97282d256dab) to create the Microsoft list.
2. Amend the JSON for list view formatting
    2.1 Amend the additional JSON for to display percentage of each slice within the doughnut

    ```Json
    {
      "elmType": "div",
      "style": {
        "display": "=if([$VALUE8.displayValue]=='0' || [$VALUE8.displayValue]=='0,0' || [$VALUE8.displayValue]=='0.0' || [$VALUE8.displayValue]=='0,00' || [$VALUE8.displayValue]=='0.00' || [$VALUE8.displayValue]=='0,000' || [$VALUE8.displayValue]=='0.000' || [$VALUE8.displayValue]=='0.0000' || [$VALUE8.displayValue]=='0,0000' || [$VALUE8.displayValue]=='0,00000' || [$VALUE8.displayValue]=='0.00000', 'none', 'flex')",
        "justify-content": "center",
        "align-items": "flex-start",
        "height": "20px",
        "width": "100%"
      },
      "children": [
        {
          "elmType": "div",
          "style": {
            "height": "14px",
            "width": "14px",
            "border-radius": "50%",
            "background-color": "=if([$COLOR8] =='', 'LightGrey', [$COLOR8])",
            "margin-left": "10px"
          }
        },
        {
          "elmType": "span",
          "style": {
            "height": "15px",
            "display": "flex",
            "align-items": "center",
            "width": "56px",
            "margin-left": "10px",
            "font-size": "15px"
          },
          "txtContent": "=if(substring(toString([$VALUE8]/([$VALUE1]+[$VALUE2]+[$VALUE3]+[$VALUE4]+[$VALUE5]+[$VALUE6]+[$VALUE9]+[$VALUE8]+[$VALUE7])), 2, 3) == '0', substring(toString([$VALUE8]/([$VALUE1]+[$VALUE2]+[$VALUE3]+[$VALUE4]+[$VALUE5]+[$VALUE6]+[$VALUE7]+[$VALUE8]+[$VALUE9])*100), 0, 4) + '%', substring(toString([$VALUE8]/([$VALUE1]+[$VALUE2]+[$VALUE3]+[$VALUE4]+[$VALUE5]+[$VALUE8]+[$VALUE9]+[$VALUE8]+[$VALUE8])*100), 0, 5) + '%'"
        }
      ]
    }
    ```

    This JSON object represents a div element with some conditional styling based on SharePoint column values and two children elements - a circular div element and a span element with dynamic text content. The layout is designed to be responsive with certain elements hidden or shown based on specific conditions.
   2.2 Amend the JSON to allow inline editing of the labels and values only by the author
   
    ```Json
    {
      "elmType": "div",
      "style": {
        "display": "=if([$VALUE8.displayValue]=='0' || [$VALUE8.displayValue]=='0,0' || [$VALUE8.displayValue]=='0,00' || [$VALUE8.displayValue]=='0,000' || [$VALUE8.displayValue]=='0,0000' || [$VALUE8.displayValue]=='0,00000', 'none', 'flex')",
        "justify-content": "flex-start",
        "align-items": "center",
        "margin-bottom": "4px",
        "max-width": "485px"
      },
      "children": [
        {
          "elmType": "div",
          "inlineEditField": "[$COLOR8]",
          "style": {
            "box-sizing": "border-box",
            "height": "20px",
            "width": "20px",
            "border-radius": "50%",
            "background-color": "=if([$COLOR8] =='', 'LightGrey', [$COLOR8])",
            "margin-right": "8px",
            "cursor": "=if([$Author.email] == @me, 'pointer', '')"
          },
          "attributes": {
            "title": "=if([$Author.email] == @me, 'Change color - HTML names or HEX allowed', '')"
          }
        },
        {
          "elmType": "div",
          "inlineEditField": "[$LABEL8]",
          "style": {
            "height": "40px",
            "box-sizing": "border-box",
            "display": "block",
            "padding-top": "8px",
            "max-width": "=if([$Author.email] == @me,'300px', '315px')",
            "margin-right": "8px",
            "font-size": "18px",
            "font-weight": "600",
            "color": "=if([$LABEL8] =='', '#bdbfbe', 'Grey')",
            "cursor": "=if([$Author.email] == @me, 'pointer', '')",
            "overflow": "hidden",
            "white-space": "nowrap",
            "text-overflow": "ellipsis"
          },
          "txtContent": "=if([$LABEL8] =='', 'Write a label title', [$LABEL8])",
          "attributes": {
            "title": "[$LABEL8]"
          }
        },
        {
          "elmType": "div",
          "inlineEditField": "[$VALUE8]",
          "style": {
            "height": "40px",
            "min-width": "20px",
            "max-width": "=if([$Author.email] == @me, '115px', '125px')",
            "box-sizing": "border-box",
            "display": "block",
            "text-align": "right",
            "white-space": "nowrap",
            "text-overflow": "ellipsis",
            "overflow": "hidden",
            "margin-right": "8px",
            "padding-top": "9px",
            "font-size": "17px",
            "font-weight": "bold",
            "color": "Black",
            "cursor": "=if([$Author.email] == @me, 'pointer', '')"
          },
          "attributes": {
            "title": "[$VALUE8.displayValue]"
          },
          "txtContent": "[$VALUE8.displayValue]"
        },
        {
          "elmType": "div",
          "customRowAction": {
            "action": "setValue",
            "actionInput": {
              "VALUE6": "0",
              "LABEL6": ""
            }
          },
          "style": {
            "height": "40px",
            "width": "26px",
            "box-sizing": "border-box",
            "display": "=if([$Author.email] == @me, 'block', 'none')",
            "text-align": "center",
            "padding-top": "13px",
            "font-size": "15px",
            "cursor": "pointer"
          },
          "attributes": {
            "class": "sp-css-color-GrayText ms-bgColor-severeWarning--hover ms-fontColor-black--hover",
            "iconName": "Clear",
            "title": "Delete data"
          }
        }
      ]
    }
    ```

 The Json renders four child elements within a div element
    a. Circular Color Indicator:
    This child div represents a circular color indicator.
    It allows inline editing for the [$COLOR8] column value.
    The background color depends on the [$COLOR8] value and will be "LightGrey" if empty.
    It has a cursor that changes to a pointer if the current user is the author.
    Hovering over it displays a tooltip with the option to change the color.

    b. Label Title:
    
    This child div displays a label title for the item.
    It allows inline editing for the [$LABEL8] column value.
    The content is dynamically generated based on the [$LABEL8] value.
    If [$LABEL8] is empty, the content will be "Write a label title."
    The font size, weight, and color change based on conditions.
    The div has ellipsis for long text, and a tooltip displays the [$LABEL8] value on hover.
    
    c. Value Display:
    
    This child div displays the value of the item.
    It allows inline editing for the [$VALUE8] column value.
    The content is dynamically generated based on the [$VALUE8.displayValue] value.
    The font size, weight, and color are fixed.
    The div has ellipsis for long text, and a tooltip displays the [$VALUE8.displayValue] value on hover.
    
    d. Delete Action (Visible to Author):
    
    This child div is a delete action that only appears to the author.
    It triggers a custom row action to set VALUE6 to "0" and clear LABEL6.
    It has a fixed width and height and shows a cursor pointer on hover.
    It displays an icon ("Clear") and has a tooltip indicating "Delete data."

    2.3 Amend the SVG to allow for 3 more slices, it proved a bit a little bit more challenging for someone who did not understand SVG and through trial and iteration , managed to resolve.     

  ```json
  {
    "elmType": "svg",
    "style": {
      "fill": "=if([$COLOR8] =='', 'LightGrey', [$COLOR8])",
      "display": "=if([$VALUE8]=='', 'none', '')",
      "position": "absolute",
      "z-index": "2",
      "top": "0",
      "right": "0",
      "stroke": "white"
    },
    "attributes": {
      "viewBox": "0 0 600 600"
    },
    "children": [
      {
        "elmType": "path",
        "attributes": {
          "d": "= 'M300,0' + 'A300,300' + ' 0 ' + if(([$VALUE8]+[$VALUE7]+[$VALUE6]+[$VALUE5]+[$VALUE4]+[$VALUE3]+[$VALUE2]+[$VALUE1])/([$VALUE1]+[$VALUE2]+[$VALUE3]+[$VALUE4]+[$VALUE5]+[$VALUE6]+[$VALUE7]+[$VALUE8]+[$VALUE9]) > 0.5 , '1,1 ' , '0,1 ') + (300 + 300 * sin(360 * if(([$VALUE8]+[$VALUE7]+[$VALUE6]+[$VALUE5]+[$VALUE4]+[$VALUE3]+[$VALUE2]+[$VALUE1])/([$VALUE1]+[$VALUE2]+[$VALUE3]+[$VALUE4]+[$VALUE5]+[$VALUE6]+[$VALUE9]+[$VALUE8]+[$VALUE7]) > 1 , 1 , ([$VALUE8]+[$VALUE7]+[$VALUE6]+[$VALUE5]+[$VALUE4]+[$VALUE3]+[$VALUE2]+[$VALUE1])/([$VALUE1]+[$VALUE2]+[$VALUE3]+[$VALUE4]+[$VALUE5]+[$VALUE6]+[$VALUE9]+[$VALUE8]+[$VALUE7]) * (3.14159 / 180))) + ',' + (300 - 300 * cos(360 * if(([$VALUE8]+[$VALUE7]+[$VALUE6]+[$VALUE5]+[$VALUE4]+[$VALUE3]+[$VALUE2]+[$VALUE1])/([$VALUE1]+[$VALUE2]+[$VALUE3]+[$VALUE4]+[$VALUE5]+[$VALUE6]+[$VALUE9]+[$VALUE8]+[$VALUE7]) > 1 , 1 , ([$VALUE8]+[$VALUE7]+[$VALUE6]+[$VALUE5]+[$VALUE4]+[$VALUE3]+[$VALUE2]+[$VALUE1])/([$VALUE1]+[$VALUE2]+[$VALUE3]+[$VALUE4]+[$VALUE5]+[$VALUE6]+[$VALUE9]+[$VALUE8]+[$VALUE7]) * (3.14159 / 180))) + 'L300,300' + 'Z'"
        }
      }
    ]
  }
  ```

 This JSON object represents an SVG element with a pie chart that dynamically changes based on the values of specific SharePoint columns ($VALUE1, $VALUE2, ..., $VALUE9). The pie chart's appearance depends on the [$COLOR8] value, and it will be displayed or hidden based on the presence of [$VALUE8].

The d attribute defines the path of the SVG:

"d": "= 'M300,0' + 'A300,300' + ... + 'L300,300' + 'Z'": 
The path data is constructed using expressions that calculate the coordinates for creating a circular pie chart. The path starts from the top point (12 o'clock position) with 'M300,0', indicating a move to the point (300, 0) in the SVG coordinate system.
The A command represents an elliptical arc. The arc's parameters are calculated based on various values from different columns ($VALUE1, $VALUE2, ..., $VALUE9) to form the pie chart.
The L command is used to draw a line from the last point back to the center of the circle at 'L300,300'.
The Z command closes the path, connecting the last point to the starting point, completing the circle.

The amended JSON to cater for 10 slices

```JSON
{
  "$schema": "https://developer.microsoft.com/json-schemas/sp/v2/row-formatting.schema.json",
  "hideColumnHeader": true,
  "hideSelection": true,
  "rowFormatter": {
    "elmType": "div",
    "style": {
      "display": "flex",
      "align-items": "flex-end",
      "justify-content": "center",
      "box-sizing": "border-box",
      "width": "750px",
      "height": "420px",
      "border-right": "solid 1px grey"
    },
    "children": [
      {
        "elmType": "div",
        "inlineEditField": "[$Title]",
        "style": {
          "width": "714px",
          "box-sizing": "border-box",
          "margin-right": "36px",
          "display": "flex",
          "align-items": "flex-start",
          "justify-content": "center",
          "height": "36px",
          "position": "absolute",
          "top": "0"
        },
        "children": [
          {
            "elmType": "div",
            "style": {
              "position": "relative",
              "height": "36px",
              "max-width": "714px",
              "display": "block",
              "top": "-6px",
              "box-sizing": "border-box",
              "white-space": "nowrap",
              "overflow": "hidden",
              "text-overflow": "ellipsis",
              "font-size": "20px",
              "font-weight": "bold",
              "color": "black",
              "cursor": "=if([$Author.email] == @me, 'pointer', '')"
            },
            "txtContent": "[$Title]"
          }
        ]
      },
      {
        "elmType": "div",
        "customCardProps": {
          "formatter": {
            "elmType": "div",
            "style": {
              "width": "480px",
              "overflow": "hidden",
              "display": "block",
              "box-sizing": "border-box",
              "border-radius": "6px",
              "border": "solid 2px grey",
              "background-color": "white"
            },
            "children": [
              {
                "elmType": "div",
                "style": {
                  "height": "34px",
                  "width": "100%",
                  "display": "flex",
                  "align-items": "flex-end",
                  "justify-content": "center",
                  "box-sizing": "border-box",
                  "color": "grey",
                  "font-size": "22px",
                  "font-weight": "700"
                },
                "txtContent": "Add data label"
              },
              {
                "elmType": "div",
                "style": {
                  "box-sizing": "border-box",
                  "width": "100%",
                  "height": "34px",
                  "display": "flex",
                  "align-items": "flex-end",
                  "justify-content": "center",
                  "margin": "30px 0px 30px 0px"
                },
                "children": [
                  {
                    "elmType": "div",
                    "customRowAction": {
                      "action": "setValue",
                      "actionInput": {
                        "VALUE1": "1"
                      }
                    },
                    "style": {
                      "box-sizing": "border-box",
                      "width": "80px",
                      "display": "=if([$VALUE1] != 0, 'none', 'flex')",
                      "align-items": "center",
                      "justify-content": "center",
                      "background-color": "#e8e8e8",
                      "border-radius": "16px",
                      "height": "34px",
                      "margin": "0px 4px 0px 4px",
                      "font-size": "16px",
                      "font-weight": "bold",
                      "color": "Grey",
                      "cursor": "pointer"
                    },
                    "txtContent": "Label 1"
                  },
                  {
                    "elmType": "div",
                    "customRowAction": {
                      "action": "setValue",
                      "actionInput": {
                        "VALUE2": "1"
                      }
                    },
                    "style": {
                      "box-sizing": "border-box",
                      "width": "80px",
                      "display": "=if([$VALUE2] != 0, 'none', 'flex')",
                      "align-items": "center",
                      "justify-content": "center",
                      "background-color": "#e8e8e8",
                      "border-radius": "16px",
                      "height": "34px",
                      "margin": "0px 4px 0px 4px",
                      "font-size": "16px",
                      "font-weight": "bold",
                      "color": "Grey",
                      "cursor": "pointer"
                    },
                    "txtContent": "Label 2"
                  },
                  {
                    "elmType": "div",
                    "customRowAction": {
                      "action": "setValue",
                      "actionInput": {
                        "VALUE3": "1"
                      }
                    },
                    "style": {
                      "box-sizing": "border-box",
                      "width": "80px",
                      "display": "=if([$VALUE3] != 0, 'none', 'flex')",
                      "align-items": "center",
                      "justify-content": "center",
                      "background-color": "#e8e8e8",
                      "border-radius": "16px",
                      "height": "34px",
                      "margin": "0px 4px 0px 4px",
                      "font-size": "16px",
                      "font-weight": "bold",
                      "color": "Grey",
                      "cursor": "pointer"
                    },
                    "txtContent": "Label 3"
                  },
                  {
                    "elmType": "div",
                    "customRowAction": {
                      "action": "setValue",
                      "actionInput": {
                        "VALUE4": "1"
                      }
                    },
                    "style": {
                      "box-sizing": "border-box",
                      "width": "80px",
                      "display": "=if([$VALUE4] != 0, 'none', 'flex')",
                      "align-items": "center",
                      "justify-content": "center",
                      "background-color": "#e8e8e8",
                      "border-radius": "16px",
                      "height": "34px",
                      "margin": "0px 4px 0px 4px",
                      "font-size": "16px",
                      "font-weight": "bold",
                      "color": "Grey",
                      "cursor": "pointer"
                    },
                    "txtContent": "Label 4"
                  },
                  {
                    "elmType": "div",
                    "customRowAction": {
                      "action": "setValue",
                      "actionInput": {
                        "VALUE5": "1"
                      }
                    },
                    "style": {
                      "box-sizing": "border-box",
                      "width": "80px",
                      "display": "=if([$VALUE5] != 0, 'none', 'flex')",
                      "align-items": "center",
                      "justify-content": "center",
                      "background-color": "#e8e8e8",
                      "border-radius": "16px",
                      "height": "34px",
                      "margin": "0px 4px 0px 4px",
                      "font-size": "16px",
                      "font-weight": "bold",
                      "color": "Grey",
                      "cursor": "pointer"
                    },
                    "txtContent": "Label 5"
                  },
                  {
                    "elmType": "div",
                    "customRowAction": {
                      "action": "setValue",
                      "actionInput": {
                        "VALUE6": "1"
                      }
                    },
                    "style": {
                      "box-sizing": "border-box",
                      "width": "80px",
                      "display": "=if([$VALUE6] != 0, 'none', 'flex')",
                      "align-items": "center",
                      "justify-content": "center",
                      "background-color": "#e8e8e8",
                      "border-radius": "16px",
                      "height": "34px",
                      "margin": "0px 4px 0px 4px",
                      "font-size": "16px",
                      "font-weight": "bold",
                      "color": "Grey",
                      "cursor": "pointer"
                    },
                    "txtContent": "Label 6"
                  },
                  {
                    "elmType": "div",
                    "customRowAction": {
                      "action": "setValue",
                      "actionInput": {
                        "VALUE7": "1"
                      }
                    },
                    "style": {
                      "box-sizing": "border-box",
                      "width": "80px",
                      "display": "=if([$VALUE7] != 0, 'none', 'flex')",
                      "align-items": "center",
                      "justify-content": "center",
                      "background-color": "#e8e8e8",
                      "border-radius": "16px",
                      "height": "34px",
                      "margin": "0px 4px 0px 4px",
                      "font-size": "16px",
                      "font-weight": "bold",
                      "color": "Grey",
                      "cursor": "pointer"
                    },
                    "txtContent": "Label 7"
                  },
                  {
                    "elmType": "div",
                    "customRowAction": {
                      "action": "setValue",
                      "actionInput": {
                        "VALUE8": "1"
                      }
                    },
                    "style": {
                      "box-sizing": "border-box",
                      "width": "80px",
                      "display": "=if([$VALUE8] != 0, 'none', 'flex')",
                      "align-items": "center",
                      "justify-content": "center",
                      "background-color": "#e8e8e8",
                      "border-radius": "16px",
                      "height": "34px",
                      "margin": "0px 4px 0px 4px",
                      "font-size": "16px",
                      "font-weight": "bold",
                      "color": "Grey",
                      "cursor": "pointer"
                    },
                    "txtContent": "Label 8"
                  },
                  {
                    "elmType": "div",
                    "customRowAction": {
                      "action": "setValue",
                      "actionInput": {
                        "VALUE9": "1"
                      }
                    },
                    "style": {
                      "box-sizing": "border-box",
                      "width": "80px",
                      "display": "=if([$VALUE9] != 0, 'none', 'flex')",
                      "align-items": "center",
                      "justify-content": "center",
                      "background-color": "#e8e8e8",
                      "border-radius": "16px",
                      "height": "34px",
                      "margin": "0px 4px 0px 4px",
                      "font-size": "16px",
                      "font-weight": "bold",
                      "color": "Grey",
                      "cursor": "pointer"
                    },
                    "txtContent": "Label 9"
                  }
                ]
              }
            ]
          },
          "openOnEvent": "click",
          "directionalHint": "leftCenter",
          "isBeakVisible": true,
          "beakStyle": {
            "backgroundColor": "white"
          }
        },
        "style": {
          "position": "absolute",
          "top": "0",
          "left": "720px",
          "height": "36px",
          "width": "26px",
          "box-sizing": "border-box",
          "display": "block",
          "visibility": "=if([$Author.email] == @me, 'visible', 'hidden')",
          "text-align": "center",
          "font-size": "16px",
          "font-weight": "bold",
          "cursor": "pointer"
        },
        "attributes": {
          "class": "sp-css-color-GrayText",
          "iconName": "MoreVertical"
        }
      },
      {
        "elmType": "div",
        "style": {
          "width": "250px",
          "height": "264px",
          "margin-right": "15px",
          "display": "flex",
          "align-items": "center"
        },
        "children": [
          {
            "elmType": "div",
            "style": {
              "position": "relative",
              "width": "250px",
              "height": "250px",
              "display": "flex",
              "align-items": "center",
              "justify-content": "center"
            },
            "children": [
              {
                "elmType": "svg",
                "style": {
                  "fill": "currentColor",
                  "top": "0",
                  "right": "0"
                },
                "attributes": {
                  "viewBox": "0 0 600 600",
                  "class": "ms-fontColor-neutralQuaternaryAlt"
                },
                "children": [
                  {
                    "elmType": "path",
                    "attributes": {
                      "d": "M300,0 A300,300 0 1,1 299.998407846124, 4.2249439502484165e-9 L299.99880588459297,75.00000000316868 A225,225 0 1,0 300,75 Z"
                    }
                  }
                ]
              },
              {
                "elmType": "svg",
                "style": {
                  "fill": "=if([$COLOR3] =='', 'LightGrey', [$COLOR3])",
                  "display": "=if([$VALUE3]=='', 'none', '')",
                  "position": "absolute",
                  "z-index": "7",
                  "top": "0",
                  "right": "0",
                  "stroke": "white"
                },
                "attributes": {
                  "viewBox": "0 0 600 600"
                },
                "children": [
                  {
                    "elmType": "path",
                    "attributes": {
                      "d": "= 'M300,0' + 'A300,300' + ' 0 ' + if(([$VALUE3]+[$VALUE2]+[$VALUE1])/([$VALUE1]+[$VALUE2]+[$VALUE3]+[$VALUE4]+[$VALUE5]+[$VALUE6]+[$VALUE7]+[$VALUE8]+[$VALUE9]) > 0.5 , '1,1 ' , '0,1 ') + (300 + 300 * sin(360 * if(([$VALUE3]+[$VALUE2]+[$VALUE1])/([$VALUE1]+[$VALUE2]+[$VALUE3]+[$VALUE4]+[$VALUE5]+[$VALUE6]+[$VALUE9]+[$VALUE8]+[$VALUE7]) > 1 , 1 , ([$VALUE3]+[$VALUE2]+[$VALUE1])/([$VALUE1]+[$VALUE2]+[$VALUE3]+[$VALUE4]+[$VALUE5]+[$VALUE6]+[$VALUE9]+[$VALUE8]+[$VALUE7]) * (3.14159 / 180))) + ',' + (300 - 300 * cos(360 * if(([$VALUE3]+[$VALUE2]+[$VALUE1])/([$VALUE1]+[$VALUE2]+[$VALUE3]+[$VALUE4]+[$VALUE5]+[$VALUE6]+[$VALUE9]+[$VALUE8]+[$VALUE7]) > 1 , 1 , ([$VALUE3]+[$VALUE2]+[$VALUE1])/([$VALUE1]+[$VALUE2]+[$VALUE3]+[$VALUE4]+[$VALUE5]+[$VALUE6]+[$VALUE9]+[$VALUE8]+[$VALUE7]) * (3.14159 / 180))) + 'L300,300' + 'Z'"
                    }
                  }
                ]
              },
              {
                "elmType": "svg",
                "style": {
                  "fill": "=if([$COLOR1] =='', 'LightGrey', [$COLOR1])",
                  "display": "=if([$VALUE1]=='', 'none', '')",
                  "position": "absolute",
                  "z-index": "9",
                  "top": "0",
                  "right": "0",
                  "stroke": "white"
                },
                "attributes": {
                  "viewBox": "0 0 600 600"
                },
                "children": [
                  {
                    "elmType": "path",
                    "attributes": {
                      "d": "= 'M300,0' + 'A300,300' + ' 0 ' + if(([$VALUE1])/([$VALUE1]+[$VALUE2]+[$VALUE3]+[$VALUE4]+[$VALUE5]+[$VALUE6]+[$VALUE7]+[$VALUE8]+[$VALUE9]) > 0.5 , '1,1 ' , '0,1 ') + (300 + 300 * sin(360 * if(([$VALUE1])/([$VALUE1]+[$VALUE2]+[$VALUE3]+[$VALUE4]+[$VALUE5]+[$VALUE6]+[$VALUE9]+[$VALUE8]+[$VALUE7]) > 1 , 1 , ([$VALUE1])/([$VALUE1]+[$VALUE2]+[$VALUE3]+[$VALUE4]+[$VALUE5]+[$VALUE6]+[$VALUE9]+[$VALUE8]+[$VALUE7]) * (3.14159 / 180))) + ',' + (300 - 300 * cos(360 * if(([$VALUE1])/([$VALUE1]+[$VALUE2]+[$VALUE3]+[$VALUE4]+[$VALUE5]+[$VALUE6]+[$VALUE9]+[$VALUE8]+[$VALUE7]) > 1 , 1 , ([$VALUE1])/([$VALUE1]+[$VALUE2]+[$VALUE3]+[$VALUE4]+[$VALUE5]+[$VALUE6]+[$VALUE9]+[$VALUE8]+[$VALUE7]) * (3.14159 / 180))) + 'L300,300' + 'Z'"
                    }
                  }
                ]
              },
              {
                "elmType": "svg",
                "style": {
                  "fill": "=if([$COLOR2] =='', 'LightGrey', [$COLOR2])",
                  "display": "=if([$VALUE2]=='', 'none', '')",
                  "position": "absolute",
                  "z-index": "8",
                  "top": "0",
                  "right": "0",
                  "stroke": "white"
                },
                "attributes": {
                  "viewBox": "0 0 600 600"
                },
                "children": [
                  {
                    "elmType": "path",
                    "attributes": {
                      "d": "= 'M300,0' + 'A300,300' + ' 0 ' + if(([$VALUE2]+[$VALUE1])/([$VALUE1]+[$VALUE2]+[$VALUE3]+[$VALUE4]+[$VALUE5]+[$VALUE6]+[$VALUE7]+[$VALUE8]+[$VALUE9]) > 0.5 , '1,1 ' , '0,1 ') + (300 + 300 * sin(360 * if(([$VALUE2]+[$VALUE1])/([$VALUE1]+[$VALUE2]+[$VALUE3]+[$VALUE4]+[$VALUE5]+[$VALUE6]+[$VALUE9]+[$VALUE8]+[$VALUE7]) > 1 , 1 , ([$VALUE2]+[$VALUE1])/([$VALUE1]+[$VALUE2]+[$VALUE3]+[$VALUE4]+[$VALUE5]+[$VALUE6]+[$VALUE9]+[$VALUE8]+[$VALUE7]) * (3.14159 / 180))) + ',' + (300 - 300 * cos(360 * if(([$VALUE2]+[$VALUE1])/([$VALUE1]+[$VALUE2]+[$VALUE3]+[$VALUE4]+[$VALUE5]+[$VALUE6]+[$VALUE9]+[$VALUE8]+[$VALUE7]) > 1 , 1 , ([$VALUE2]+[$VALUE1])/([$VALUE1]+[$VALUE2]+[$VALUE3]+[$VALUE4]+[$VALUE5]+[$VALUE6]+[$VALUE9]+[$VALUE8]+[$VALUE7]) * (3.14159 / 180))) + 'L300,300' + 'Z'"
                    }
                  }
                ]
              },
              {
                "elmType": "svg",
                "style": {
                  "fill": "=if([$COLOR4] =='', 'LightGrey', [$COLOR4])",
                  "display": "=if([$VALUE4]=='', 'none', '')",
                  "position": "absolute",
                  "z-index": "6",
                  "top": "0",
                  "right": "0",
                  "stroke": "white"
                },
                "attributes": {
                  "viewBox": "0 0 600 600"
                },
                "children": [
                  {
                    "elmType": "path",
                    "attributes": {
                      "d": "= 'M300,0' + 'A300,300' + ' 0 ' + if(([$VALUE4]+[$VALUE3]+[$VALUE2]+[$VALUE1])/([$VALUE1]+[$VALUE2]+[$VALUE3]+[$VALUE4]+[$VALUE5]+[$VALUE6]+[$VALUE7]+[$VALUE8]+[$VALUE9]) > 0.5 , '1,1 ' , '0,1 ') + (300 + 300 * sin(360 * if(([$VALUE4]+[$VALUE3]+[$VALUE2]+[$VALUE1])/([$VALUE1]+[$VALUE2]+[$VALUE3]+[$VALUE4]+[$VALUE5]+[$VALUE6]+[$VALUE9]+[$VALUE8]+[$VALUE7]) > 1 , 1 , ([$VALUE4]+[$VALUE3]+[$VALUE2]+[$VALUE1])/([$VALUE1]+[$VALUE2]+[$VALUE3]+[$VALUE4]+[$VALUE5]+[$VALUE6]+[$VALUE9]+[$VALUE8]+[$VALUE7]) * (3.14159 / 180))) + ',' + (300 - 300 * cos(360 * if(([$VALUE4]+[$VALUE3]+[$VALUE2]+[$VALUE1])/([$VALUE1]+[$VALUE2]+[$VALUE3]+[$VALUE4]+[$VALUE5]+[$VALUE6]+[$VALUE9]+[$VALUE8]+[$VALUE7]) > 1 , 1 , ([$VALUE4]+[$VALUE3]+[$VALUE2]+[$VALUE1])/([$VALUE1]+[$VALUE2]+[$VALUE3]+[$VALUE4]+[$VALUE5]+[$VALUE6]+[$VALUE9]+[$VALUE8]+[$VALUE7]) * (3.14159 / 180))) + 'L300,300' + 'Z'"
                    }
                  }
                ]
              },
              {
                "elmType": "svg",
                "style": {
                  "fill": "=if([$COLOR5] =='', 'LightGrey', [$COLOR5])",
                  "display": "=if([$VALUE5]=='', 'none', '')",
                  "position": "absolute",
                  "z-index": "5",
                  "top": "0",
                  "right": "0",
                  "stroke": "white"
                },
                "attributes": {
                  "viewBox": "0 0 600 600"
                },
                "children": [
                  {
                    "elmType": "path",
                    "attributes": {
                      "d": "= 'M300,0' + 'A300,300' + ' 0 ' + if(([$VALUE5]+[$VALUE4]+[$VALUE3]+[$VALUE2]+[$VALUE1])/([$VALUE1]+[$VALUE2]+[$VALUE3]+[$VALUE4]+[$VALUE5]+[$VALUE6]+[$VALUE7]+[$VALUE8]+[$VALUE9]) > 0.5 , '1,1 ' , '0,1 ') + (300 + 300 * sin(360 * if(([$VALUE5]+[$VALUE4]+[$VALUE3]+[$VALUE2]+[$VALUE1])/([$VALUE1]+[$VALUE2]+[$VALUE3]+[$VALUE4]+[$VALUE5]+[$VALUE6]+[$VALUE9]+[$VALUE8]+[$VALUE7]) > 1 , 1 , ([$VALUE5]+[$VALUE4]+[$VALUE3]+[$VALUE2]+[$VALUE1])/([$VALUE1]+[$VALUE2]+[$VALUE3]+[$VALUE4]+[$VALUE5]+[$VALUE6]+[$VALUE9]+[$VALUE8]+[$VALUE7]) * (3.14159 / 180))) + ',' + (300 - 300 * cos(360 * if(([$VALUE5]+[$VALUE4]+[$VALUE3]+[$VALUE2]+[$VALUE1])/([$VALUE1]+[$VALUE2]+[$VALUE3]+[$VALUE4]+[$VALUE5]+[$VALUE6]+[$VALUE9]+[$VALUE8]+[$VALUE7]) > 1 , 1 , ([$VALUE5]+[$VALUE4]+[$VALUE3]+[$VALUE2]+[$VALUE1])/([$VALUE1]+[$VALUE2]+[$VALUE3]+[$VALUE4]+[$VALUE5]+[$VALUE6]+[$VALUE9]+[$VALUE8]+[$VALUE7]) * (3.14159 / 180))) + 'L300,300' + 'Z'"
                    }
                  }
                ]
              },
              {
                "elmType": "svg",
                "style": {
                  "fill": "=if([$COLOR6] =='', 'LightGrey', [$COLOR6])",
                  "display": "=if([$VALUE6]=='', 'none', '')",
                  "position": "absolute",
                  "z-index": "4",
                  "top": "0",
                  "right": "0",
                  "stroke": "white"
                },
                "attributes": {
                  "viewBox": "0 0 600 600"
                },
                "children": [
                  {
                    "elmType": "path",
                    "attributes": {
                      "d": "= 'M300,0' + 'A300,300' + ' 0 ' + if(([$VALUE6]+[$VALUE5]+[$VALUE4]+[$VALUE3]+[$VALUE2]+[$VALUE1])/([$VALUE1]+[$VALUE2]+[$VALUE3]+[$VALUE4]+[$VALUE5]+[$VALUE6]+[$VALUE7]+[$VALUE8]+[$VALUE9]) > 0.5 , '1,1 ' , '0,1 ') + (300 + 300 * sin(360 * if(([$VALUE6]+[$VALUE5]+[$VALUE4]+[$VALUE3]+[$VALUE2]+[$VALUE1])/([$VALUE1]+[$VALUE2]+[$VALUE3]+[$VALUE4]+[$VALUE5]+[$VALUE6]+[$VALUE9]+[$VALUE8]+[$VALUE7]) > 1 , 1 , ([$VALUE6]+[$VALUE5]+[$VALUE4]+[$VALUE3]+[$VALUE2]+[$VALUE1])/([$VALUE1]+[$VALUE2]+[$VALUE3]+[$VALUE4]+[$VALUE5]+[$VALUE6]+[$VALUE9]+[$VALUE8]+[$VALUE7]) * (3.14159 / 180))) + ',' + (300 - 300 * cos(360 * if(([$VALUE6]+[$VALUE5]+[$VALUE4]+[$VALUE3]+[$VALUE2]+[$VALUE1])/([$VALUE1]+[$VALUE2]+[$VALUE3]+[$VALUE4]+[$VALUE5]+[$VALUE6]+[$VALUE9]+[$VALUE8]+[$VALUE7]) > 1 , 1 , ([$VALUE6]+[$VALUE5]+[$VALUE4]+[$VALUE3]+[$VALUE2]+[$VALUE1])/([$VALUE1]+[$VALUE2]+[$VALUE3]+[$VALUE4]+[$VALUE5]+[$VALUE6]+[$VALUE9]+[$VALUE8]+[$VALUE7]) * (3.14159 / 180))) + 'L300,300' + 'Z'"
                    }
                  }
                ]
              },
              {
                "elmType": "svg",
                "style": {
                  "fill": "=if([$COLOR7] =='', 'LightGrey', [$COLOR7])",
                  "display": "=if([$VALUE7]=='', 'none', '')",
                  "position": "absolute",
                  "z-index": "3",
                  "top": "0",
                  "right": "0",
                  "stroke": "white"
                },
                "attributes": {
                  "viewBox": "0 0 600 600"
                },
                "children": [
                  {
                    "elmType": "path",
                    "attributes": {
                      "d": "= 'M300,0' + 'A300,300' + ' 0 ' + if(([$VALUE7]+[$VALUE6]+[$VALUE5]+[$VALUE4]+[$VALUE3]+[$VALUE2]+[$VALUE1])/([$VALUE1]+[$VALUE2]+[$VALUE3]+[$VALUE4]+[$VALUE5]+[$VALUE6]+[$VALUE7]+[$VALUE8]+[$VALUE9]) > 0.5 , '1,1 ' , '0,1 ') + (300 + 300 * sin(360 * if(([$VALUE7]+[$VALUE6]+[$VALUE5]+[$VALUE4]+[$VALUE3]+[$VALUE2]+[$VALUE1])/([$VALUE1]+[$VALUE2]+[$VALUE3]+[$VALUE4]+[$VALUE5]+[$VALUE6]+[$VALUE9]+[$VALUE8]+[$VALUE7]) > 1 , 1 , ([$VALUE7]+[$VALUE6]+[$VALUE5]+[$VALUE4]+[$VALUE3]+[$VALUE2]+[$VALUE1])/([$VALUE1]+[$VALUE2]+[$VALUE3]+[$VALUE4]+[$VALUE5]+[$VALUE6]+[$VALUE9]+[$VALUE8]+[$VALUE7]) * (3.14159 / 180))) + ',' + (300 - 300 * cos(360 * if(([$VALUE7]+[$VALUE6]+[$VALUE5]+[$VALUE4]+[$VALUE3]+[$VALUE2]+[$VALUE1])/([$VALUE1]+[$VALUE2]+[$VALUE3]+[$VALUE4]+[$VALUE5]+[$VALUE6]+[$VALUE9]+[$VALUE8]+[$VALUE7]) > 1 , 1 , ([$VALUE7]+[$VALUE6]+[$VALUE5]+[$VALUE4]+[$VALUE3]+[$VALUE2]+[$VALUE1])/([$VALUE1]+[$VALUE2]+[$VALUE3]+[$VALUE4]+[$VALUE5]+[$VALUE6]+[$VALUE9]+[$VALUE8]+[$VALUE7]) * (3.14159 / 180))) + 'L300,300' + 'Z'"
                    }
                  }
                ]
              },
              {
                "elmType": "svg",
                "style": {
                  "fill": "=if([$COLOR8] =='', 'LightGrey', [$COLOR8])",
                  "display": "=if([$VALUE8]=='', 'none', '')",
                  "position": "absolute",
                  "z-index": "2",
                  "top": "0",
                  "right": "0",
                  "stroke": "white"
                },
                "attributes": {
                  "viewBox": "0 0 600 600"
                },
                "children": [
                  {
                    "elmType": "path",
                    "attributes": {
                      "d": "= 'M300,0' + 'A300,300' + ' 0 ' + if(([$VALUE8]+[$VALUE7]+[$VALUE6]+[$VALUE5]+[$VALUE4]+[$VALUE3]+[$VALUE2]+[$VALUE1])/([$VALUE1]+[$VALUE2]+[$VALUE3]+[$VALUE4]+[$VALUE5]+[$VALUE6]+[$VALUE7]+[$VALUE8]+[$VALUE9]) > 0.5 , '1,1 ' , '0,1 ') + (300 + 300 * sin(360 * if(([$VALUE8]+[$VALUE7]+[$VALUE6]+[$VALUE5]+[$VALUE4]+[$VALUE3]+[$VALUE2]+[$VALUE1])/([$VALUE1]+[$VALUE2]+[$VALUE3]+[$VALUE4]+[$VALUE5]+[$VALUE6]+[$VALUE9]+[$VALUE8]+[$VALUE7]) > 1 , 1 , ([$VALUE8]+[$VALUE7]+[$VALUE6]+[$VALUE5]+[$VALUE4]+[$VALUE3]+[$VALUE2]+[$VALUE1])/([$VALUE1]+[$VALUE2]+[$VALUE3]+[$VALUE4]+[$VALUE5]+[$VALUE6]+[$VALUE9]+[$VALUE8]+[$VALUE7]) * (3.14159 / 180))) + ',' + (300 - 300 * cos(360 * if(([$VALUE8]+[$VALUE7]+[$VALUE6]+[$VALUE5]+[$VALUE4]+[$VALUE3]+[$VALUE2]+[$VALUE1])/([$VALUE1]+[$VALUE2]+[$VALUE3]+[$VALUE4]+[$VALUE5]+[$VALUE6]+[$VALUE9]+[$VALUE8]+[$VALUE7]) > 1 , 1 , ([$VALUE8]+[$VALUE7]+[$VALUE6]+[$VALUE5]+[$VALUE4]+[$VALUE3]+[$VALUE2]+[$VALUE1])/([$VALUE1]+[$VALUE2]+[$VALUE3]+[$VALUE4]+[$VALUE5]+[$VALUE6]+[$VALUE9]+[$VALUE8]+[$VALUE7]) * (3.14159 / 180))) + 'L300,300' + 'Z'"
                    }
                  }
                ]
              },
              {
                "elmType": "svg",
                "style": {
                  "fill": "=if([$COLOR9] =='', 'LightGrey', [$COLOR9])",
                  "display": "=if([$VALUE9]=='', 'none', '')",
                  "position": "absolute",
                  "z-index": "1",
                  "top": "0",
                  "right": "0",
                  "stroke": "white"
                },
                "attributes": {
                  "viewBox": "0 0 600 600"
                },
                "children": [
                  {
                    "elmType": "path",
                    "attributes": {
                      "d": "= 'M300,0' + 'A300,300' + ' 0 ' + if(([$VALUE9]+[$VALUE8]+[$VALUE7]+[$VALUE6]+[$VALUE5]+[$VALUE4]+[$VALUE3]+[$VALUE2]+[$VALUE1])/([$VALUE1]+[$VALUE2]+[$VALUE3]+[$VALUE4]+[$VALUE5]+[$VALUE6]+[$VALUE7]+[$VALUE8]+[$VALUE9]) > 0.5 , '1,1 ' , '0,1 ') + (300 + 300 * sin(360 * if(([$VALUE9]+[$VALUE8]+[$VALUE7]+[$VALUE6]+[$VALUE5]+[$VALUE4]+[$VALUE3]+[$VALUE2]+[$VALUE1])/([$VALUE1]+[$VALUE2]+[$VALUE3]+[$VALUE4]+[$VALUE5]+[$VALUE6]+[$VALUE9]+[$VALUE8]+[$VALUE7]) > 1 , 1 , ([$VALUE9]+[$VALUE8]+[$VALUE7]+[$VALUE6]+[$VALUE5]+[$VALUE4]+[$VALUE3]+[$VALUE2]+[$VALUE1])/([$VALUE1]+[$VALUE2]+[$VALUE3]+[$VALUE4]+[$VALUE5]+[$VALUE6]+[$VALUE9]+[$VALUE8]+[$VALUE7]) * (3.14159 / 180))) + ',' + (300 - 300 * cos(360 * if(([$VALUE9]+[$VALUE8]+[$VALUE7]+[$VALUE6]+[$VALUE5]+[$VALUE4]+[$VALUE3]+[$VALUE2]+[$VALUE1])/([$VALUE1]+[$VALUE2]+[$VALUE3]+[$VALUE4]+[$VALUE5]+[$VALUE6]+[$VALUE9]+[$VALUE8]+[$VALUE7]) > 1 , 1 , ([$VALUE9]+[$VALUE8]+[$VALUE7]+[$VALUE6]+[$VALUE5]+[$VALUE4]+[$VALUE3]+[$VALUE2]+[$VALUE1])/([$VALUE1]+[$VALUE2]+[$VALUE3]+[$VALUE4]+[$VALUE5]+[$VALUE6]+[$VALUE9]+[$VALUE8]+[$VALUE7]) * (3.14159 / 180))) + 'L300,300' + 'Z'"
                    }
                  }
                ]
              },
              {
                "elmType": "div",
                "style": {
                  "position": "absolute",
                  "z-index": "10",
                  "width": "70%",
                  "height": "70%",
                  "display": "flex",
                  "justify-content": "center",
                  "align-items": "center",
                  "background-color": "white",
                  "border-radius": "50%"
                },
                "attributes": {
                  "class": "sp-field-bold"
                },
                "children": [
                  {
                    "elmType": "div",
                    "style": {
                      "box-sizing": "border-box",
                      "display": "flex",
                      "align-items": "center",
                      "align-content": "center",
                      "flex-wrap": "wrap",
                      "width": "90px"
                    },
                    "children": [
                      {
                        "elmType": "div",
                        "style": {
                          "display": "=if([$VALUE1.displayValue]=='0' || [$VALUE1.displayValue]=='0,0' || [$VALUE1.displayValue]=='0.0' || [$VALUE1.displayValue]=='0,00' || [$VALUE1.displayValue]=='0.00' || [$VALUE1.displayValue]=='0,000' || [$VALUE1.displayValue]=='0.000' || [$VALUE1.displayValue]=='0.0000' || [$VALUE1.displayValue]=='0,0000' || [$VALUE1.displayValue]=='0,00000' || [$VALUE1.displayValue]=='0.00000', 'none', 'flex')",
                          "justify-content": "center",
                          "align-items": "flex-start",
                          "height": "20px",
                          "width": "100%"
                        },
                        "children": [
                          {
                            "elmType": "div",
                            "style": {
                              "height": "14px",
                              "width": "14px",
                              "border-radius": "50%",
                              "background-color": "=if([$COLOR1] =='', 'LightGrey', [$COLOR1])",
                              "margin-left": "10px"
                            }
                          },
                          {
                            "elmType": "span",
                            "style": {
                              "height": "15px",
                              "display": "flex",
                              "align-items": "center",
                              "width": "56px",
                              "margin-left": "10px",
                              "font-size": "15px"
                            },
                            "txtContent": "=if(substring(toString([$VALUE1]/([$VALUE1]+[$VALUE2]+[$VALUE3]+[$VALUE4]+[$VALUE5]+[$VALUE6]+[$VALUE9]+[$VALUE8]+[$VALUE7])), 2, 3) == '0', substring(toString([$VALUE1]/([$VALUE1]+[$VALUE2]+[$VALUE3]+[$VALUE4]+[$VALUE5]+[$VALUE6]+[$VALUE9]+[$VALUE8]+[$VALUE7])*100), 0, 4) + '%', substring(toString([$VALUE1]/([$VALUE1]+[$VALUE2]+[$VALUE3]+[$VALUE4]+[$VALUE5]+[$VALUE6]+[$VALUE9]+[$VALUE8]+[$VALUE7])*100), 0, 5) + '%'"
                          }
                        ]
                      },
                      {
                        "elmType": "div",
                        "style": {
                          "display": "=if([$VALUE2.displayValue]=='0' || [$VALUE2.displayValue]=='0,0' || [$VALUE2.displayValue]=='0.0' || [$VALUE2.displayValue]=='0,00' || [$VALUE2.displayValue]=='0.00' || [$VALUE2.displayValue]=='0,000' || [$VALUE2.displayValue]=='0.000' || [$VALUE2.displayValue]=='0.0000' || [$VALUE2.displayValue]=='0,0000' || [$VALUE2.displayValue]=='0,00000' || [$VALUE2.displayValue]=='0.00000', 'none', 'flex')",
                          "justify-content": "center",
                          "align-items": "flex-start",
                          "height": "20px",
                          "width": "100%"
                        },
                        "children": [
                          {
                            "elmType": "div",
                            "style": {
                              "height": "14px",
                              "width": "14px",
                              "border-radius": "50%",
                              "background-color": "=if([$COLOR2] =='', 'LightGrey', [$COLOR2])",
                              "margin-left": "10px"
                            }
                          },
                          {
                            "elmType": "span",
                            "style": {
                              "height": "15px",
                              "display": "flex",
                              "align-items": "center",
                              "width": "56px",
                              "margin-left": "10px",
                              "font-size": "15px"
                            },
                            "txtContent": "=if(substring(toString([$VALUE2]/([$VALUE1]+[$VALUE2]+[$VALUE3]+[$VALUE4]+[$VALUE5]+[$VALUE6]+[$VALUE9]+[$VALUE8]+[$VALUE7])), 2, 3) == '0', substring(toString([$VALUE2]/([$VALUE1]+[$VALUE2]+[$VALUE3]+[$VALUE4]+[$VALUE5]+[$VALUE6]+[$VALUE9]+[$VALUE8]+[$VALUE7])*100), 0, 4) + '%', substring(toString([$VALUE2]/([$VALUE1]+[$VALUE2]+[$VALUE3]+[$VALUE4]+[$VALUE5]+[$VALUE6]+[$VALUE9]+[$VALUE8]+[$VALUE7])*100), 0, 5) + '%'"
                          }
                        ]
                      },
                      {
                        "elmType": "div",
                        "style": {
                          "display": "=if([$VALUE3.displayValue]=='0' || [$VALUE3.displayValue]=='0,0' || [$VALUE3.displayValue]=='0.0' || [$VALUE3.displayValue]=='0,00' || [$VALUE3.displayValue]=='0.00' || [$VALUE3.displayValue]=='0,000' || [$VALUE3.displayValue]=='0.000' || [$VALUE3.displayValue]=='0.0000' || [$VALUE3.displayValue]=='0,0000' || [$VALUE3.displayValue]=='0,00000' || [$VALUE3.displayValue]=='0.00000', 'none', 'flex')",
                          "justify-content": "center",
                          "align-items": "flex-start",
                          "height": "20px",
                          "width": "100%"
                        },
                        "children": [
                          {
                            "elmType": "div",
                            "style": {
                              "height": "14px",
                              "width": "14px",
                              "border-radius": "50%",
                              "background-color": "=if([$COLOR3] =='', 'LightGrey', [$COLOR3])",
                              "margin-left": "10px"
                            }
                          },
                          {
                            "elmType": "span",
                            "style": {
                              "height": "15px",
                              "display": "flex",
                              "align-items": "center",
                              "width": "56px",
                              "margin-left": "10px",
                              "font-size": "15px"
                            },
                            "txtContent": "=if(substring(toString([$VALUE3]/([$VALUE1]+[$VALUE2]+[$VALUE3]+[$VALUE4]+[$VALUE5]+[$VALUE6]+[$VALUE9]+[$VALUE8]+[$VALUE7])), 2, 3) == '0', substring(toString([$VALUE3]/([$VALUE1]+[$VALUE2]+[$VALUE3]+[$VALUE4]+[$VALUE5]+[$VALUE6]+[$VALUE9]+[$VALUE8]+[$VALUE7])*100), 0, 4) + '%', substring(toString([$VALUE3]/([$VALUE1]+[$VALUE2]+[$VALUE3]+[$VALUE4]+[$VALUE5]+[$VALUE6]+[$VALUE9]+[$VALUE8]+[$VALUE7])*100), 0, 5) + '%'"
                          }
                        ]
                      },
                      {
                        "elmType": "div",
                        "style": {
                          "display": "=if([$VALUE4.displayValue]=='0' || [$VALUE4.displayValue]=='0,0' || [$VALUE4.displayValue]=='0.0' || [$VALUE4.displayValue]=='0,00' || [$VALUE4.displayValue]=='0.00' || [$VALUE4.displayValue]=='0,000' || [$VALUE4.displayValue]=='0.000' || [$VALUE4.displayValue]=='0.0000' || [$VALUE4.displayValue]=='0,0000' || [$VALUE4.displayValue]=='0,00000' || [$VALUE4.displayValue]=='0.00000', 'none', 'flex')",
                          "justify-content": "center",
                          "align-items": "flex-start",
                          "height": "20px",
                          "width": "100%"
                        },
                        "children": [
                          {
                            "elmType": "div",
                            "style": {
                              "height": "14px",
                              "width": "14px",
                              "border-radius": "50%",
                              "background-color": "=if([$COLOR4] =='', 'LightGrey', [$COLOR4])",
                              "margin-left": "10px"
                            }
                          },
                          {
                            "elmType": "span",
                            "style": {
                              "height": "15px",
                              "display": "flex",
                              "align-items": "center",
                              "width": "56px",
                              "margin-left": "10px",
                              "font-size": "15px"
                            },
                            "txtContent": "=if(substring(toString([$VALUE4]/([$VALUE1]+[$VALUE2]+[$VALUE3]+[$VALUE4]+[$VALUE5]+[$VALUE6]+[$VALUE9]+[$VALUE8]+[$VALUE7])), 2, 3) == '0', substring(toString([$VALUE4]/([$VALUE1]+[$VALUE2]+[$VALUE3]+[$VALUE4]+[$VALUE5]+[$VALUE6]+[$VALUE9]+[$VALUE8]+[$VALUE7])*100), 0, 4) + '%', substring(toString([$VALUE4]/([$VALUE1]+[$VALUE2]+[$VALUE3]+[$VALUE4]+[$VALUE5]+[$VALUE6]+[$VALUE9]+[$VALUE8]+[$VALUE7])*100), 0, 5) + '%'"
                          }
                        ]
                      },
                      {
                        "elmType": "div",
                        "style": {
                          "display": "=if([$VALUE5.displayValue]=='0' || [$VALUE5.displayValue]=='0,0' || [$VALUE5.displayValue]=='0.0' || [$VALUE5.displayValue]=='0,00' || [$VALUE5.displayValue]=='0.00' || [$VALUE5.displayValue]=='0,000' || [$VALUE5.displayValue]=='0.000' || [$VALUE5.displayValue]=='0.0000' || [$VALUE5.displayValue]=='0,0000' || [$VALUE5.displayValue]=='0,00000' || [$VALUE5.displayValue]=='0.00000', 'none', 'flex')",
                          "justify-content": "center",
                          "align-items": "flex-start",
                          "height": "20px",
                          "width": "100%"
                        },
                        "children": [
                          {
                            "elmType": "div",
                            "style": {
                              "height": "14px",
                              "width": "14px",
                              "border-radius": "50%",
                              "background-color": "=if([$COLOR5] =='', 'LightGrey', [$COLOR5])",
                              "margin-left": "10px"
                            }
                          },
                          {
                            "elmType": "span",
                            "style": {
                              "height": "15px",
                              "display": "flex",
                              "align-items": "center",
                              "width": "56px",
                              "margin-left": "10px",
                              "font-size": "15px"
                            },
                            "txtContent": "=if(substring(toString([$VALUE5]/([$VALUE1]+[$VALUE2]+[$VALUE3]+[$VALUE4]+[$VALUE5]+[$VALUE6]+[$VALUE9]+[$VALUE8]+[$VALUE7])), 2, 3) == '0', substring(toString([$VALUE5]/([$VALUE1]+[$VALUE2]+[$VALUE3]+[$VALUE4]+[$VALUE5]+[$VALUE6]+[$VALUE9]+[$VALUE8]+[$VALUE7])*100), 0, 4) + '%', substring(toString([$VALUE5]/([$VALUE1]+[$VALUE2]+[$VALUE3]+[$VALUE4]+[$VALUE5]+[$VALUE6]+[$VALUE9]+[$VALUE8]+[$VALUE7])*100), 0, 5) + '%'"
                          }
                        ]
                      },
                      {
                        "elmType": "div",
                        "style": {
                          "display": "=if([$VALUE6.displayValue]=='0' || [$VALUE6.displayValue]=='0,0' || [$VALUE6.displayValue]=='0.0' || [$VALUE6.displayValue]=='0,00' || [$VALUE6.displayValue]=='0.00' || [$VALUE6.displayValue]=='0,000' || [$VALUE6.displayValue]=='0.000' || [$VALUE6.displayValue]=='0.0000' || [$VALUE6.displayValue]=='0,0000' || [$VALUE6.displayValue]=='0,00000' || [$VALUE6.displayValue]=='0.00000', 'none', 'flex')",
                          "justify-content": "center",
                          "align-items": "flex-start",
                          "height": "20px",
                          "width": "100%"
                        },
                        "children": [
                          {
                            "elmType": "div",
                            "style": {
                              "height": "14px",
                              "width": "14px",
                              "border-radius": "50%",
                              "background-color": "=if([$COLOR6] =='', 'LightGrey', [$COLOR6])",
                              "margin-left": "10px"
                            }
                          },
                          {
                            "elmType": "span",
                            "style": {
                              "height": "15px",
                              "display": "flex",
                              "align-items": "center",
                              "width": "56px",
                              "margin-left": "10px",
                              "font-size": "15px"
                            },
                            "txtContent": "=if(substring(toString([$VALUE6]/([$VALUE1]+[$VALUE2]+[$VALUE3]+[$VALUE4]+[$VALUE5]+[$VALUE6]+[$VALUE7]+[$VALUE8]+[$VALUE9])), 2, 3) == '0', substring(toString([$VALUE6]/([$VALUE1]+[$VALUE2]+[$VALUE3]+[$VALUE4]+[$VALUE5]+[$VALUE6]+[$VALUE9]+[$VALUE8]+[$VALUE7])*100), 0, 4) + '%', substring(toString([$VALUE6]/([$VALUE1]+[$VALUE2]+[$VALUE3]+[$VALUE4]+[$VALUE5]+[$VALUE6]+[$VALUE9]+[$VALUE8]+[$VALUE7])*100), 0, 5) + '%'"
                          }
                        ]
                      },
                      {
                        "elmType": "div",
                        "style": {
                          "display": "=if([$VALUE7.displayValue]=='0' || [$VALUE7.displayValue]=='0,0' || [$VALUE7.displayValue]=='0.0' || [$VALUE7.displayValue]=='0,00' || [$VALUE7.displayValue]=='0.00' || [$VALUE7.displayValue]=='0,000' || [$VALUE7.displayValue]=='0.000' || [$VALUE7.displayValue]=='0.0000' || [$VALUE7.displayValue]=='0,0000' || [$VALUE7.displayValue]=='0,00000' || [$VALUE7.displayValue]=='0.00000', 'none', 'flex')",
                          "justify-content": "center",
                          "align-items": "flex-start",
                          "height": "20px",
                          "width": "100%"
                        },
                        "children": [
                          {
                            "elmType": "div",
                            "style": {
                              "height": "14px",
                              "width": "14px",
                              "border-radius": "50%",
                              "background-color": "=if([$COLOR7] =='', 'LightGrey', [$COLOR7])",
                              "margin-left": "10px"
                            }
                          },
                          {
                            "elmType": "span",
                            "style": {
                              "height": "15px",
                              "display": "flex",
                              "align-items": "center",
                              "width": "56px",
                              "margin-left": "10px",
                              "font-size": "15px"
                            },
                            "txtContent": "=if(substring(toString([$VALUE7]/([$VALUE1]+[$VALUE2]+[$VALUE3]+[$VALUE4]+[$VALUE5]+[$VALUE6]+[$VALUE7]+[$VALUE8]+[$VALUE9])), 2, 3) == '0', substring(toString([$VALUE7]/([$VALUE1]+[$VALUE2]+[$VALUE3]+[$VALUE4]+[$VALUE5]+[$VALUE7]+[$VALUE9]+[$VALUE8]+[$VALUE7])*100), 0, 4) + '%', substring(toString([$VALUE7]/([$VALUE1]+[$VALUE2]+[$VALUE3]+[$VALUE4]+[$VALUE5]+[$VALUE7]+[$VALUE9]+[$VALUE8]+[$VALUE7])*100), 0, 5) + '%'"
                          }
                        ]
                      },
                      {
                        "elmType": "div",
                        "style": {
                          "display": "=if([$VALUE8.displayValue]=='0' || [$VALUE8.displayValue]=='0,0' || [$VALUE8.displayValue]=='0.0' || [$VALUE8.displayValue]=='0,00' || [$VALUE8.displayValue]=='0.00' || [$VALUE8.displayValue]=='0,000' || [$VALUE8.displayValue]=='0.000' || [$VALUE8.displayValue]=='0.0000' || [$VALUE8.displayValue]=='0,0000' || [$VALUE8.displayValue]=='0,00000' || [$VALUE8.displayValue]=='0.00000', 'none', 'flex')",
                          "justify-content": "center",
                          "align-items": "flex-start",
                          "height": "20px",
                          "width": "100%"
                        },
                        "children": [
                          {
                            "elmType": "div",
                            "style": {
                              "height": "14px",
                              "width": "14px",
                              "border-radius": "50%",
                              "background-color": "=if([$COLOR8] =='', 'LightGrey', [$COLOR8])",
                              "margin-left": "10px"
                            }
                          },
                          {
                            "elmType": "span",
                            "style": {
                              "height": "15px",
                              "display": "flex",
                              "align-items": "center",
                              "width": "56px",
                              "margin-left": "10px",
                              "font-size": "15px"
                            },
                            "txtContent": "=if(substring(toString([$VALUE8]/([$VALUE1]+[$VALUE2]+[$VALUE3]+[$VALUE4]+[$VALUE5]+[$VALUE6]+[$VALUE9]+[$VALUE8]+[$VALUE7])), 2, 3) == '0', substring(toString([$VALUE8]/([$VALUE1]+[$VALUE2]+[$VALUE3]+[$VALUE4]+[$VALUE5]+[$VALUE6]+[$VALUE7]+[$VALUE8]+[$VALUE9])*100), 0, 4) + '%', substring(toString([$VALUE8]/([$VALUE1]+[$VALUE2]+[$VALUE3]+[$VALUE4]+[$VALUE5]+[$VALUE8]+[$VALUE9]+[$VALUE8]+[$VALUE8])*100), 0, 5) + '%'"
                          }
                        ]
                      },
                      {
                        "elmType": "div",
                        "style": {
                          "display": "=if([$VALUE9.displayValue]=='0' || [$VALUE9.displayValue]=='0,0' || [$VALUE9.displayValue]=='0.0' || [$VALUE9.displayValue]=='0,00' || [$VALUE9.displayValue]=='0.00' || [$VALUE9.displayValue]=='0,000' || [$VALUE9.displayValue]=='0.000' || [$VALUE9.displayValue]=='0.0000' || [$VALUE9.displayValue]=='0,0000' || [$VALUE9.displayValue]=='0,00000' || [$VALUE9.displayValue]=='0.00000', 'none', 'flex')",
                          "justify-content": "center",
                          "align-items": "flex-start",
                          "height": "20px",
                          "width": "100%"
                        },
                        "children": [
                          {
                            "elmType": "div",
                            "style": {
                              "height": "14px",
                              "width": "14px",
                              "border-radius": "50%",
                              "background-color": "=if([$COLOR9] =='', 'LightGrey', [$COLOR9])",
                              "margin-left": "10px"
                            }
                          },
                          {
                            "elmType": "span",
                            "style": {
                              "height": "15px",
                              "display": "flex",
                              "align-items": "center",
                              "width": "56px",
                              "margin-left": "10px",
                              "font-size": "15px"
                            },
                            "txtContent": "=if(substring(toString([$VALUE9]/([$VALUE1]+[$VALUE2]+[$VALUE3]+[$VALUE4]+[$VALUE5]+[$VALUE6]+[$VALUE9]+[$VALUE8]+[$VALUE7])), 2, 3) == '0', substring(toString([$VALUE9]/([$VALUE1]+[$VALUE2]+[$VALUE3]+[$VALUE4]+[$VALUE5]+[$VALUE6]+[$VALUE7]+[$VALUE8]+[$VALUE9])*100), 0, 4) + '%', substring(toString([$VALUE9]/([$VALUE1]+[$VALUE2]+[$VALUE3]+[$VALUE4]+[$VALUE5]+[$VALUE6]+[$VALUE7]+[$VALUE7]+[$VALUE9])*100), 0, 5) + '%'"
                          }
                        ]
                      }
                    ]
                  }
                ]
              }
            ]
          }
        ]
      },
      {
        "elmType": "div",
        "style": {
          "max-width": "485px",
          "height": "264px",
          "box-sizing": "border-box",
          "display": "flex",
          "align-items": "flex-end",
          "align-content": "center",
          "flex-direction": "column",
          "justify-content": "center"
        },
        "children": [
          {
            "elmType": "div",
            "style": {
              "display": "=if([$VALUE1.displayValue]=='0' || [$VALUE1.displayValue]=='0,0' || [$VALUE1.displayValue]=='0,00' || [$VALUE1.displayValue]=='0,000' || [$VALUE1.displayValue]=='0,0000' || [$VALUE1.displayValue]=='0,00000', 'none', 'flex')",
              "justify-content": "flex-start",
              "align-items": "center",
              "margin-bottom": "4px",
              "max-width": "485px"
            },
            "children": [
              {
                "elmType": "div",
                "inlineEditField": "[$COLOR1]",
                "style": {
                  "box-sizing": "border-box",
                  "height": "20px",
                  "width": "20px",
                  "border-radius": "50%",
                  "background-color": "=if([$COLOR1] =='', 'LightGrey', [$COLOR1])",
                  "margin-right": "8px",
                  "cursor": "=if([$Author.email] == @me, 'pointer', '')"
                },
                "attributes": {
                  "title": "=if([$Author.email] == @me, 'Change color - HTML names or HEX allowed', '')"
                }
              },
              {
                "elmType": "div",
                "inlineEditField": "[$LABEL1]",
                "style": {
                  "height": "40px",
                  "box-sizing": "border-box",
                  "display": "block",
                  "padding-top": "8px",
                  "max-width": "=if([$Author.email] == @me,'300px', '315px')",
                  "margin-right": "8px",
                  "font-size": "18px",
                  "font-weight": "600",
                  "color": "=if([$LABEL1] =='', '#bdbfbe', 'Gray')",
                  "cursor": "=if([$Author.email] == @me, 'pointer', '')",
                  "overflow": "hidden",
                  "white-space": "nowrap",
                  "text-overflow": "ellipsis"
                },
                "txtContent": "=if([$LABEL1] =='', 'Write a label title', [$LABEL1])",
                "attributes": {
                  "title": "[$LABEL1]"
                }
              },
              {
                "elmType": "div",
                "inlineEditField": "[$VALUE1]",
                "style": {
                  "height": "40px",
                  "min-width": "20px",
                  "max-width": "=if([$Author.email] == @me, '115px', '125px')",
                  "box-sizing": "border-box",
                  "display": "block",
                  "text-align": "right",
                  "white-space": "nowrap",
                  "text-overflow": "ellipsis",
                  "overflow": "hidden",
                  "margin-right": "8px",
                  "padding-top": "9px",
                  "font-size": "17px",
                  "font-weight": "bold",
                  "color": "Black",
                  "cursor": "=if([$Author.email] == @me, 'pointer', '')"
                },
                "attributes": {
                  "class": "sp-css-color-GrayText",
                  "title": "[$VALUE1.displayValue]"
                },
                "txtContent": "[$VALUE1.displayValue]"
              },
              {
                "elmType": "div",
                "customRowAction": {
                  "action": "setValue",
                  "actionInput": {
                    "VALUE1": "0",
                    "LABEL1": ""
                  }
                },
                "style": {
                  "height": "40px",
                  "width": "26px",
                  "box-sizing": "border-box",
                  "display": "=if([$Author.email] == @me, 'block', 'none')",
                  "text-align": "center",
                  "padding-top": "13px",
                  "font-size": "15px",
                  "cursor": "pointer"
                },
                "attributes": {
                  "class": "sp-css-color-GrayText ms-bgColor-severeWarning--hover ms-fontColor-black--hover",
                  "iconName": "Clear",
                  "title": "Delete data"
                }
              }
            ]
          },
          {
            "elmType": "div",
            "style": {
              "display": "=if([$VALUE2.displayValue]=='0' || [$VALUE2.displayValue]=='0,0' || [$VALUE2.displayValue]=='0,00' || [$VALUE2.displayValue]=='0,000' || [$VALUE2.displayValue]=='0,0000' || [$VALUE2.displayValue]=='0,00000', 'none', 'flex')",
              "justify-content": "flex-start",
              "align-items": "center",
              "margin-bottom": "4px",
              "max-width": "485px"
            },
            "children": [
              {
                "elmType": "div",
                "inlineEditField": "[$COLOR2]",
                "style": {
                  "box-sizing": "border-box",
                  "height": "20px",
                  "width": "20px",
                  "border-radius": "50%",
                  "background-color": "=if([$COLOR2] =='', 'LightGrey', [$COLOR2])",
                  "margin-right": "8px",
                  "cursor": "=if([$Author.email] == @me, 'pointer', '')"
                },
                "attributes": {
                  "title": "=if([$Author.email] == @me, 'Change color - HTML names or HEX allowed', '')"
                }
              },
              {
                "elmType": "div",
                "inlineEditField": "[$LABEL2]",
                "style": {
                  "height": "40px",
                  "box-sizing": "border-box",
                  "display": "block",
                  "padding-top": "8px",
                  "max-width": "=if([$Author.email] == @me,'300px', '315px')",
                  "margin-right": "8px",
                  "font-size": "18px",
                  "font-weight": "600",
                  "color": "=if([$LABEL2] =='', '#bdbfbe', 'Gray')",
                  "cursor": "=if([$Author.email] == @me, 'pointer', '')",
                  "overflow": "hidden",
                  "white-space": "nowrap",
                  "text-overflow": "ellipsis"
                },
                "txtContent": "=if([$LABEL2] =='', 'Write a label title', [$LABEL2])",
                "attributes": {
                  "title": "[$LABEL2]"
                }
              },
              {
                "elmType": "div",
                "inlineEditField": "[$VALUE2]",
                "style": {
                  "height": "40px",
                  "min-width": "20px",
                  "max-width": "=if([$Author.email] == @me, '115px', '125px')",
                  "box-sizing": "border-box",
                  "display": "block",
                  "text-align": "right",
                  "white-space": "nowrap",
                  "text-overflow": "ellipsis",
                  "overflow": "hidden",
                  "margin-right": "8px",
                  "padding-top": "9px",
                  "font-size": "17px",
                  "font-weight": "bold",
                  "color": "Black",
                  "cursor": "=if([$Author.email] == @me, 'pointer', '')"
                },
                "attributes": {
                  "title": "[$VALUE2.displayValue]"
                },
                "txtContent": "[$VALUE2.displayValue]"
              },
              {
                "elmType": "div",
                "customRowAction": {
                  "action": "setValue",
                  "actionInput": {
                    "VALUE2": "0",
                    "LABEL2": ""
                  }
                },
                "style": {
                  "height": "40px",
                  "width": "26px",
                  "box-sizing": "border-box",
                  "display": "=if([$Author.email] == @me, 'block', 'none')",
                  "text-align": "center",
                  "padding-top": "13px",
                  "font-size": "15px",
                  "cursor": "pointer"
                },
                "attributes": {
                  "class": "sp-css-color-GrayText ms-bgColor-severeWarning--hover ms-fontColor-black--hover",
                  "iconName": "Clear",
                  "title": "Delete data"
                }
              }
            ]
          },
          {
            "elmType": "div",
            "style": {
              "display": "=if([$VALUE3.displayValue]=='0' || [$VALUE3.displayValue]=='0,0' || [$VALUE3.displayValue]=='0,00' || [$VALUE3.displayValue]=='0,000' || [$VALUE3.displayValue]=='0,0000' || [$VALUE3.displayValue]=='0,00000', 'none', 'flex')",
              "justify-content": "flex-start",
              "align-items": "center",
              "margin-bottom": "4px",
              "max-width": "485px"
            },
            "children": [
              {
                "elmType": "div",
                "inlineEditField": "[$COLOR3]",
                "style": {
                  "box-sizing": "border-box",
                  "height": "20px",
                  "width": "20px",
                  "border-radius": "50%",
                  "background-color": "=if([$COLOR3] =='', 'LightGrey', [$COLOR3])",
                  "margin-right": "8px",
                  "cursor": "=if([$Author.email] == @me, 'pointer', '')"
                },
                "attributes": {
                  "title": "=if([$Author.email] == @me, 'Change color - HTML names or HEX allowed', '')"
                }
              },
              {
                "elmType": "div",
                "inlineEditField": "[$LABEL3]",
                "style": {
                  "height": "40px",
                  "box-sizing": "border-box",
                  "display": "block",
                  "padding-top": "8px",
                  "max-width": "=if([$Author.email] == @me,'300px', '315px')",
                  "margin-right": "8px",
                  "font-size": "18px",
                  "font-weight": "600",
                  "color": "=if([$LABEL3] =='', '#bdbfbe', 'Grey')",
                  "cursor": "=if([$Author.email] == @me, 'pointer', '')",
                  "overflow": "hidden",
                  "white-space": "nowrap",
                  "text-overflow": "ellipsis"
                },
                "txtContent": "=if([$LABEL3] =='', 'Write a label title', [$LABEL3])",
                "attributes": {
                  "title": "[$LABEL3]"
                }
              },
              {
                "elmType": "div",
                "inlineEditField": "[$VALUE3]",
                "style": {
                  "height": "40px",
                  "min-width": "20px",
                  "max-width": "=if([$Author.email] == @me, '115px', '125px')",
                  "box-sizing": "border-box",
                  "display": "block",
                  "text-align": "right",
                  "white-space": "nowrap",
                  "text-overflow": "ellipsis",
                  "overflow": "hidden",
                  "margin-right": "8px",
                  "padding-top": "9px",
                  "font-size": "17px",
                  "font-weight": "bold",
                  "color": "Black",
                  "cursor": "=if([$Author.email] == @me, 'pointer', '')"
                },
                "attributes": {
                  "title": "[$VALUE3.displayValue]"
                },
                "txtContent": "[$VALUE3.displayValue]"
              },
              {
                "elmType": "div",
                "customRowAction": {
                  "action": "setValue",
                  "actionInput": {
                    "VALUE3": "0",
                    "LABEL3": ""
                  }
                },
                "style": {
                  "height": "40px",
                  "width": "26px",
                  "box-sizing": "border-box",
                  "display": "=if([$Author.email] == @me, 'block', 'none')",
                  "text-align": "center",
                  "padding-top": "13px",
                  "font-size": "15px",
                  "cursor": "pointer"
                },
                "attributes": {
                  "class": "sp-css-color-GrayText ms-bgColor-severeWarning--hover ms-fontColor-black--hover",
                  "iconName": "Clear",
                  "title": "Delete data"
                }
              }
            ]
          },
          {
            "elmType": "div",
            "style": {
              "display": "=if([$VALUE4.displayValue]=='0' || [$VALUE4.displayValue]=='0,0' || [$VALUE4.displayValue]=='0,00' || [$VALUE4.displayValue]=='0,000' || [$VALUE4.displayValue]=='0,0000' || [$VALUE4.displayValue]=='0,00000', 'none', 'flex')",
              "justify-content": "flex-start",
              "align-items": "center",
              "margin-bottom": "4px",
              "max-width": "485px"
            },
            "children": [
              {
                "elmType": "div",
                "inlineEditField": "[$COLOR4]",
                "style": {
                  "box-sizing": "border-box",
                  "height": "20px",
                  "width": "20px",
                  "border-radius": "50%",
                  "background-color": "=if([$COLOR4] =='', 'LightGrey', [$COLOR4])",
                  "margin-right": "8px",
                  "cursor": "=if([$Author.email] == @me, 'pointer', '')"
                },
                "attributes": {
                  "title": "=if([$Author.email] == @me, 'Change color - HTML names or HEX allowed', '')"
                }
              },
              {
                "elmType": "div",
                "inlineEditField": "[$LABEL4]",
                "style": {
                  "height": "40px",
                  "box-sizing": "border-box",
                  "display": "block",
                  "padding-top": "8px",
                  "max-width": "=if([$Author.email] == @me,'300px', '315px')",
                  "margin-right": "8px",
                  "font-size": "18px",
                  "font-weight": "600",
                  "color": "=if([$LABEL4] =='', '#bdbfbe', 'Grey')",
                  "cursor": "=if([$Author.email] == @me, 'pointer', '')",
                  "overflow": "hidden",
                  "white-space": "nowrap",
                  "text-overflow": "ellipsis"
                },
                "txtContent": "=if([$LABEL4] =='', 'Write a label title', [$LABEL4])",
                "attributes": {
                  "title": "[$LABEL4]"
                }
              },
              {
                "elmType": "div",
                "inlineEditField": "[$VALUE4]",
                "style": {
                  "height": "40px",
                  "min-width": "20px",
                  "max-width": "=if([$Author.email] == @me, '115px', '125px')",
                  "box-sizing": "border-box",
                  "display": "block",
                  "text-align": "right",
                  "white-space": "nowrap",
                  "text-overflow": "ellipsis",
                  "overflow": "hidden",
                  "margin-right": "8px",
                  "padding-top": "9px",
                  "font-size": "17px",
                  "font-weight": "bold",
                  "color": "Black",
                  "cursor": "=if([$Author.email] == @me, 'pointer', '')"
                },
                "attributes": {
                  "title": "[$VALUE4.displayValue]"
                },
                "txtContent": "[$VALUE4.displayValue]"
              },
              {
                "elmType": "div",
                "customRowAction": {
                  "action": "setValue",
                  "actionInput": {
                    "VALUE4": "0",
                    "LABEL4": ""
                  }
                },
                "style": {
                  "height": "40px",
                  "width": "26px",
                  "box-sizing": "border-box",
                  "display": "=if([$Author.email] == @me, 'block', 'none')",
                  "text-align": "center",
                  "padding-top": "13px",
                  "font-size": "15px",
                  "cursor": "pointer"
                },
                "attributes": {
                  "class": "sp-css-color-GrayText ms-bgColor-severeWarning--hover ms-fontColor-black--hover",
                  "iconName": "Clear",
                  "title": "Delete data"
                }
              }
            ]
          },
          {
            "elmType": "div",
            "style": {
              "display": "=if([$VALUE5.displayValue]=='0' || [$VALUE5.displayValue]=='0,0' || [$VALUE5.displayValue]=='0,00' || [$VALUE5.displayValue]=='0,000' || [$VALUE5.displayValue]=='0,0000' || [$VALUE5.displayValue]=='0,00000', 'none', 'flex')",
              "justify-content": "flex-start",
              "align-items": "center",
              "margin-bottom": "4px",
              "max-width": "485px"
            },
            "children": [
              {
                "elmType": "div",
                "inlineEditField": "[$COLOR5]",
                "style": {
                  "box-sizing": "border-box",
                  "height": "20px",
                  "width": "20px",
                  "border-radius": "50%",
                  "background-color": "=if([$COLOR5] =='', 'LightGrey', [$COLOR5])",
                  "margin-right": "8px",
                  "cursor": "=if([$Author.email] == @me, 'pointer', '')"
                },
                "attributes": {
                  "title": "=if([$Author.email] == @me, 'Change color - HTML names or HEX allowed', '')"
                }
              },
              {
                "elmType": "div",
                "inlineEditField": "[$LABEL5]",
                "style": {
                  "height": "40px",
                  "box-sizing": "border-box",
                  "display": "block",
                  "padding-top": "8px",
                  "max-width": "=if([$Author.email] == @me,'300px', '315px')",
                  "margin-right": "8px",
                  "font-size": "18px",
                  "font-weight": "600",
                  "color": "=if([$LABEL5] =='', '#bdbfbe', 'Grey')",
                  "cursor": "=if([$Author.email] == @me, 'pointer', '')",
                  "overflow": "hidden",
                  "white-space": "nowrap",
                  "text-overflow": "ellipsis"
                },
                "txtContent": "=if([$LABEL5] =='', 'Write a label title', [$LABEL5])",
                "attributes": {
                  "title": "[$LABEL5]"
                }
              },
              {
                "elmType": "div",
                "inlineEditField": "[$VALUE5]",
                "style": {
                  "height": "40px",
                  "min-width": "20px",
                  "max-width": "=if([$Author.email] == @me, '115px', '125px')",
                  "box-sizing": "border-box",
                  "display": "block",
                  "text-align": "right",
                  "white-space": "nowrap",
                  "text-overflow": "ellipsis",
                  "overflow": "hidden",
                  "margin-right": "8px",
                  "padding-top": "9px",
                  "font-size": "17px",
                  "font-weight": "bold",
                  "color": "Black",
                  "cursor": "=if([$Author.email] == @me, 'pointer', '')"
                },
                "attributes": {
                  "title": "[$VALUE5.displayValue]"
                },
                "txtContent": "[$VALUE5.displayValue]"
              },
              {
                "elmType": "div",
                "customRowAction": {
                  "action": "setValue",
                  "actionInput": {
                    "VALUE5": "0",
                    "LABEL5": ""
                  }
                },
                "style": {
                  "height": "40px",
                  "width": "26px",
                  "box-sizing": "border-box",
                  "display": "=if([$Author.email] == @me, 'block', 'none')",
                  "text-align": "center",
                  "padding-top": "13px",
                  "font-size": "15px",
                  "cursor": "pointer"
                },
                "attributes": {
                  "class": "sp-css-color-GrayText ms-bgColor-severeWarning--hover ms-fontColor-black--hover",
                  "iconName": "Clear",
                  "title": "Delete data"
                }
              }
            ]
          },
          {
            "elmType": "div",
            "style": {
              "display": "=if([$VALUE6.displayValue]=='0' || [$VALUE6.displayValue]=='0,0' || [$VALUE6.displayValue]=='0,00' || [$VALUE6.displayValue]=='0,000' || [$VALUE6.displayValue]=='0,0000' || [$VALUE6.displayValue]=='0,00000', 'none', 'flex')",
              "justify-content": "flex-start",
              "align-items": "center",
              "margin-bottom": "4px",
              "max-width": "485px"
            },
            "children": [
              {
                "elmType": "div",
                "inlineEditField": "[$COLOR6]",
                "style": {
                  "box-sizing": "border-box",
                  "height": "20px",
                  "width": "20px",
                  "border-radius": "50%",
                  "background-color": "=if([$COLOR6] =='', 'LightGrey', [$COLOR6])",
                  "margin-right": "8px",
                  "cursor": "=if([$Author.email] == @me, 'pointer', '')"
                },
                "attributes": {
                  "title": "=if([$Author.email] == @me, 'Change color - HTML names or HEX allowed', '')"
                }
              },
              {
                "elmType": "div",
                "inlineEditField": "[$LABEL6]",
                "style": {
                  "height": "40px",
                  "box-sizing": "border-box",
                  "display": "block",
                  "padding-top": "8px",
                  "max-width": "=if([$Author.email] == @me,'300px', '315px')",
                  "margin-right": "8px",
                  "font-size": "18px",
                  "font-weight": "600",
                  "color": "=if([$LABEL6] =='', '#bdbfbe', 'Grey')",
                  "cursor": "=if([$Author.email] == @me, 'pointer', '')",
                  "overflow": "hidden",
                  "white-space": "nowrap",
                  "text-overflow": "ellipsis"
                },
                "txtContent": "=if([$LABEL6] =='', 'Write a label title', [$LABEL6])",
                "attributes": {
                  "title": "[$LABEL6]"
                }
              },
              {
                "elmType": "div",
                "inlineEditField": "[$VALUE6]",
                "style": {
                  "height": "40px",
                  "min-width": "20px",
                  "max-width": "=if([$Author.email] == @me, '115px', '125px')",
                  "box-sizing": "border-box",
                  "display": "block",
                  "text-align": "right",
                  "white-space": "nowrap",
                  "text-overflow": "ellipsis",
                  "overflow": "hidden",
                  "margin-right": "8px",
                  "padding-top": "9px",
                  "font-size": "17px",
                  "font-weight": "bold",
                  "color": "Black",
                  "cursor": "=if([$Author.email] == @me, 'pointer', '')"
                },
                "attributes": {
                  "title": "[$VALUE6.displayValue]"
                },
                "txtContent": "[$VALUE6.displayValue]"
              },
              {
                "elmType": "div",
                "customRowAction": {
                  "action": "setValue",
                  "actionInput": {
                    "VALUE6": "0",
                    "LABEL6": ""
                  }
                },
                "style": {
                  "height": "40px",
                  "width": "26px",
                  "box-sizing": "border-box",
                  "display": "=if([$Author.email] == @me, 'block', 'none')",
                  "text-align": "center",
                  "padding-top": "13px",
                  "font-size": "15px",
                  "cursor": "pointer"
                },
                "attributes": {
                  "class": "sp-css-color-GrayText ms-bgColor-severeWarning--hover ms-fontColor-black--hover",
                  "iconName": "Clear",
                  "title": "Delete data"
                }
              }
            ]
          },
          {
            "elmType": "div",
            "style": {
              "display": "=if([$VALUE7.displayValue]=='0' || [$VALUE7.displayValue]=='0,0' || [$VALUE7.displayValue]=='0,00' || [$VALUE7.displayValue]=='0,000' || [$VALUE7.displayValue]=='0,0000' || [$VALUE7.displayValue]=='0,00000', 'none', 'flex')",
              "justify-content": "flex-start",
              "align-items": "center",
              "margin-bottom": "4px",
              "max-width": "485px"
            },
            "children": [
              {
                "elmType": "div",
                "inlineEditField": "[$COLOR7]",
                "style": {
                  "box-sizing": "border-box",
                  "height": "20px",
                  "width": "20px",
                  "border-radius": "50%",
                  "background-color": "=if([$COLOR7] =='', 'LightGrey', [$COLOR7])",
                  "margin-right": "8px",
                  "cursor": "=if([$Author.email] == @me, 'pointer', '')"
                },
                "attributes": {
                  "title": "=if([$Author.email] == @me, 'Change color - HTML names or HEX allowed', '')"
                }
              },
              {
                "elmType": "div",
                "inlineEditField": "[$LABEL7]",
                "style": {
                  "height": "40px",
                  "box-sizing": "border-box",
                  "display": "block",
                  "padding-top": "8px",
                  "max-width": "=if([$Author.email] == @me,'300px', '315px')",
                  "margin-right": "8px",
                  "font-size": "18px",
                  "font-weight": "600",
                  "color": "=if([$LABEL7] =='', '#bdbfbe', 'Grey')",
                  "cursor": "=if([$Author.email] == @me, 'pointer', '')",
                  "overflow": "hidden",
                  "white-space": "nowrap",
                  "text-overflow": "ellipsis"
                },
                "txtContent": "=if([$LABEL7] =='', 'Write a label title', [$LABEL7])",
                "attributes": {
                  "title": "[$LABEL7]"
                }
              },
              {
                "elmType": "div",
                "inlineEditField": "[$VALUE7]",
                "style": {
                  "height": "40px",
                  "min-width": "20px",
                  "max-width": "=if([$Author.email] == @me, '115px', '125px')",
                  "box-sizing": "border-box",
                  "display": "block",
                  "text-align": "right",
                  "white-space": "nowrap",
                  "text-overflow": "ellipsis",
                  "overflow": "hidden",
                  "margin-right": "8px",
                  "padding-top": "9px",
                  "font-size": "17px",
                  "font-weight": "bold",
                  "color": "Black",
                  "cursor": "=if([$Author.email] == @me, 'pointer', '')"
                },
                "attributes": {
                  "title": "[$VALUE7.displayValue]"
                },
                "txtContent": "[$VALUE7.displayValue]"
              },
              {
                "elmType": "div",
                "customRowAction": {
                  "action": "setValue",
                  "actionInput": {
                    "VALUE6": "0",
                    "LABEL6": ""
                  }
                },
                "style": {
                  "height": "40px",
                  "width": "26px",
                  "box-sizing": "border-box",
                  "display": "=if([$Author.email] == @me, 'block', 'none')",
                  "text-align": "center",
                  "padding-top": "13px",
                  "font-size": "15px",
                  "cursor": "pointer"
                },
                "attributes": {
                  "class": "sp-css-color-GrayText ms-bgColor-severeWarning--hover ms-fontColor-black--hover",
                  "iconName": "Clear",
                  "title": "Delete data"
                }
              }
            ]
          },
          {
            "elmType": "div",
            "style": {
              "display": "=if([$VALUE8.displayValue]=='0' || [$VALUE8.displayValue]=='0,0' || [$VALUE8.displayValue]=='0,00' || [$VALUE8.displayValue]=='0,000' || [$VALUE8.displayValue]=='0,0000' || [$VALUE8.displayValue]=='0,00000', 'none', 'flex')",
              "justify-content": "flex-start",
              "align-items": "center",
              "margin-bottom": "4px",
              "max-width": "485px"
            },
            "children": [
              {
                "elmType": "div",
                "inlineEditField": "[$COLOR8]",
                "style": {
                  "box-sizing": "border-box",
                  "height": "20px",
                  "width": "20px",
                  "border-radius": "50%",
                  "background-color": "=if([$COLOR8] =='', 'LightGrey', [$COLOR8])",
                  "margin-right": "8px",
                  "cursor": "=if([$Author.email] == @me, 'pointer', '')"
                },
                "attributes": {
                  "title": "=if([$Author.email] == @me, 'Change color - HTML names or HEX allowed', '')"
                }
              },
              {
                "elmType": "div",
                "inlineEditField": "[$LABEL8]",
                "style": {
                  "height": "40px",
                  "box-sizing": "border-box",
                  "display": "block",
                  "padding-top": "8px",
                  "max-width": "=if([$Author.email] == @me,'300px', '315px')",
                  "margin-right": "8px",
                  "font-size": "18px",
                  "font-weight": "600",
                  "color": "=if([$LABEL8] =='', '#bdbfbe', 'Grey')",
                  "cursor": "=if([$Author.email] == @me, 'pointer', '')",
                  "overflow": "hidden",
                  "white-space": "nowrap",
                  "text-overflow": "ellipsis"
                },
                "txtContent": "=if([$LABEL8] =='', 'Write a label title', [$LABEL8])",
                "attributes": {
                  "title": "[$LABEL8]"
                }
              },
              {
                "elmType": "div",
                "inlineEditField": "[$VALUE8]",
                "style": {
                  "height": "40px",
                  "min-width": "20px",
                  "max-width": "=if([$Author.email] == @me, '115px', '125px')",
                  "box-sizing": "border-box",
                  "display": "block",
                  "text-align": "right",
                  "white-space": "nowrap",
                  "text-overflow": "ellipsis",
                  "overflow": "hidden",
                  "margin-right": "8px",
                  "padding-top": "9px",
                  "font-size": "17px",
                  "font-weight": "bold",
                  "color": "Black",
                  "cursor": "=if([$Author.email] == @me, 'pointer', '')"
                },
                "attributes": {
                  "title": "[$VALUE8.displayValue]"
                },
                "txtContent": "[$VALUE8.displayValue]"
              },
              {
                "elmType": "div",
                "customRowAction": {
                  "action": "setValue",
                  "actionInput": {
                    "VALUE6": "0",
                    "LABEL6": ""
                  }
                },
                "style": {
                  "height": "40px",
                  "width": "26px",
                  "box-sizing": "border-box",
                  "display": "=if([$Author.email] == @me, 'block', 'none')",
                  "text-align": "center",
                  "padding-top": "13px",
                  "font-size": "15px",
                  "cursor": "pointer"
                },
                "attributes": {
                  "class": "sp-css-color-GrayText ms-bgColor-severeWarning--hover ms-fontColor-black--hover",
                  "iconName": "Clear",
                  "title": "Delete data"
                }
              }
            ]
          },
          {
            "elmType": "div",
            "style": {
              "display": "=if([$VALUE9.displayValue]=='0' || [$VALUE9.displayValue]=='0,0' || [$VALUE9.displayValue]=='0,00' || [$VALUE9.displayValue]=='0,000' || [$VALUE9.displayValue]=='0,0000' || [$VALUE9.displayValue]=='0,00000', 'none', 'flex')",
              "justify-content": "flex-start",
              "align-items": "center",
              "margin-bottom": "4px",
              "max-width": "485px"
            },
            "children": [
              {
                "elmType": "div",
                "inlineEditField": "[$COLOR9]",
                "style": {
                  "box-sizing": "border-box",
                  "height": "20px",
                  "width": "20px",
                  "border-radius": "50%",
                  "background-color": "=if([$COLOR9] =='', 'LightGrey', [$COLOR9])",
                  "margin-right": "8px",
                  "cursor": "=if([$Author.email] == @me, 'pointer', '')"
                },
                "attributes": {
                  "title": "=if([$Author.email] == @me, 'Change color - HTML names or HEX allowed', '')"
                }
              },
              {
                "elmType": "div",
                "inlineEditField": "[$LABEL9]",
                "style": {
                  "height": "40px",
                  "box-sizing": "border-box",
                  "display": "block",
                  "padding-top": "8px",
                  "max-width": "=if([$Author.email] == @me,'300px', '315px')",
                  "margin-right": "8px",
                  "font-size": "18px",
                  "font-weight": "600",
                  "color": "=if([$LABEL9] =='', '#bdbfbe', 'Grey')",
                  "cursor": "=if([$Author.email] == @me, 'pointer', '')",
                  "overflow": "hidden",
                  "white-space": "nowrap",
                  "text-overflow": "ellipsis"
                },
                "txtContent": "=if([$LABEL9] =='', 'Write a label title', [$LABEL9])",
                "attributes": {
                  "title": "[$LABEL9]"
                }
              },
              {
                "elmType": "div",
                "inlineEditField": "[$VALUE9]",
                "style": {
                  "height": "40px",
                  "min-width": "20px",
                  "max-width": "=if([$Author.email] == @me, '115px', '125px')",
                  "box-sizing": "border-box",
                  "display": "block",
                  "text-align": "right",
                  "white-space": "nowrap",
                  "text-overflow": "ellipsis",
                  "overflow": "hidden",
                  "margin-right": "8px",
                  "padding-top": "9px",
                  "font-size": "17px",
                  "font-weight": "bold",
                  "color": "Black",
                  "cursor": "=if([$Author.email] == @me, 'pointer', '')"
                },
                "attributes": {
                  "title": "[$VALUE9.displayValue]"
                },
                "txtContent": "[$VALUE9.displayValue]"
              },
              {
                "elmType": "div",
                "customRowAction": {
                  "action": "setValue",
                  "actionInput": {
                    "VALUE6": "0",
                    "LABEL6": ""
                  }
                },
                "style": {
                  "height": "40px",
                  "width": "26px",
                  "box-sizing": "border-box",
                  "display": "=if([$Author.email] == @me, 'block', 'none')",
                  "text-align": "center",
                  "padding-top": "13px",
                  "font-size": "15px",
                  "cursor": "pointer"
                },
                "attributes": {
                  "class": "sp-css-color-GrayText ms-bgColor-severeWarning--hover ms-fontColor-black--hover",
                  "iconName": "Clear",
                  "title": "Delete data"
                }
              }
            ]
          }
        ]
      }
    ]
  }
}
```
