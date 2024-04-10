**Update 2024-04-11: This page is left as a reference for how Change Analysis Change Notification Preview users could use change data for setting up alerts with Azure Monitor. We recommend all Azure users to use Azure Monitor's native support for querying the Resource Graph for alert queries on resource change data. See the [annoucement blog](https://techcommunity.microsoft.com/t5/azure-observability-blog/query-azure-resource-graph-from-azure-monitor/ba-p/3918298) and [official documentation](https://learn.microsoft.com/en-us/azure/azure-monitor/logs/azure-monitor-data-explorer-proxy) for more information.**

The following query is an example that looks at VM SKU changes that would incur cost to the subscription. Use the query to configure Azure Monitor Custom Log Alerts. Learn more about how to configure alerts based on logs at [Create, view, and manage log alerts using Azure Monitor](https://docs.microsoft.com/en-us/azure/azure-monitor/platform/alerts-log)

```kql
//************************
// Step 1: update variables that select the appropriate data
let EventSchemaVersion = "1.0";
let ResourceType = "Microsoft.Compute/virtualMachines";
let PropertyName = "properties.hardwareProfile";
let ValueName = "vmSize";
//************************
// Step 2: set up cost comparison
let CostMap = dynamic({ 
    "Basic_A0": 13.14,
    "Basic_A1": 23.36,
    "Basic_A2": 97.09,
    "Basic_A3": 216.08,
    "Basic_A4": 432.16,
    "Standard_A0":   14.60,
    "Standard_A1":   65.70,
    "Standard_A2":  131.40,
    "Standard_A3":  262.80,
    "Standard_A4":  525.60,
    "Standard_A5":  240.90,
    "Standard_A6":  481.80,
    "Standard_A7":  963.60,
    "Standard_A8":  1070.18,
    "Standard_A9":  2141.09,
    "Standard_A10": 856.29,
    "Standard_A11": 1712.58,
    "Standard_A1_v2":   47.45,
    "Standard_A2_v2":   99.28,
    "Standard_A4_v2":   208.78,
    "Standard_A8_v2":   438.00,
    "Standard_A2m_v2":  131.40,
    "Standard_A4m_v2":  262.80,
    "Standard_A8m_v2":  525.60,
    "Standard_B1s": 10.22,
    "Standard_B1ls": 10.22,
    "Standard_B1ms":    17.96,
    "Standard_B2s": 36.21,
    "Standard_B2ms":    66.58,
    "Standard_B4ms":    132.86,
    "Standard_B8ms":    266.45,
    "Standard_D1":  0,
    "Standard_D2":  0,
    "Standard_D3":  0,
    "Standard_D4":  0,
    "Standard_D11": 0,
    "Standard_D12": 0,
    "Standard_D13": 0,
    "Standard_D14": 0,
    "Standard_D1_v2":   0,
    "Standard_D2_v2":   0,
    "Standard_D3_v2":   0,
    "Standard_D4_v2":   0,
    "Standard_D5_v2":   0,
    "Standard_D2_v3":   0,
    "Standard_D4_v3":   0,
    "Standard_D8_v3":   0,
    "Standard_D16_v3":  0,
    "Standard_D32_v3":  0,
    "Standard_D64_v3":  0,
    "Standard_D2s_v3":  0,
    "Standard_D4s_v3":  0,
    "Standard_D8s_v3":  0,
    "Standard_D16s_v3": 0,
    "Standard_D32s_v3": 0,
    "Standard_D64s_v3": 0,
    "Standard_D11_v2":  0,
    "Standard_D12_v2":  0,
    "Standard_D13_v2":  0,
    "Standard_D14_v2":  0,
    "Standard_D15_v2":  0,
    "Standard_DS1": 0,
    "Standard_DS2": 0,
    "Standard_DS3": 0,
    "Standard_DS4": 0,
    "Standard_DS11":    0,
    "Standard_DS12":    0,
    "Standard_DS13":    0,
    "Standard_DS14":    0,
    "Standard_DS1_v2":  0,
    "Standard_DS2_v2":  0,
    "Standard_DS3_v2":  0,
    "Standard_DS4_v2":  0,
    "Standard_DS5_v2":  0,
    "Standard_DS11_v2": 0,
    "Standard_DS12_v2": 0,
    "Standard_DS13_v2": 0,
    "Standard_DS14_v2": 0,
    "Standard_DS15_v2": 0,
    "Standard_DS13-4_v2":   0,
    "Standard_DS13-2_v2":   0,
    "Standard_DS14-8_v2":   0,
    "Standard_DS14-4_v2":   0,
    "Standard_E2_v3":   0,
    "Standard_E4_v3":   0,
    "Standard_E8_v3":   0,
    "Standard_E16_v3":  0,
   "Standard_E32_v3":  0,
    "Standard_E64_v3":  0,
    "Standard_E2s_v3":  0,
    "Standard_E4s_v3":  0,
    "Standard_E8s_v3":  0,
    "Standard_E16s_v3": 0,
    "Standard_E32s_v3": 0,
    "Standard_E64s_v3": 0,
    "Standard_E32-16_v3":   0,
    "Standard_E32-8s_v3":   0,
    "Standard_E64-32s_v3":  0,
    "Standard_E64-16s_v3":  0,
    "Standard_F1":  0,
    "Standard_F2":  0,
    "Standard_F4":  0,
    "Standard_F8":  0,
    "Standard_F16": 0,
    "Standard_F1s": 0,
    "Standard_F2s": 0,
    "Standard_F4s": 0,
    "Standard_F8s": 0,
    "Standard_F16s":    0,
    "Standard_F2s_v2":  0,
    "Standard_F4s_v2":  0,
    "Standard_F8s_v2":  0,
    "Standard_F16s_v2": 0,
    "Standard_F32s_v2": 0,
    "Standard_F64s_v2": 0,
    "Standard_F72s_v2": 0,
    "Standard_G1":  0,
    "Standard_G2":  0,
    "Standard_G3":  0,
    "Standard_G4":  0,
    "Standard_G5":  0,
    "Standard_GS1": 0,
    "Standard_GS2": 0,
    "Standard_GS3": 0,
    "Standard_GS4": 0,
    "Standard_GS5": 0,
    "Standard_GS4-8":   0,
    "Standard_GS4-4":   0,
    "Standard_GS5-16":  0,
    "Standard_GS5-8":   0,
    "Standard_H8":  0,
    "Standard_H16": 0,
    "Standard_H8m": 0,
    "Standard_H16m":    0,
    "Standard_H16r":    0,
    "Standard_H16mr":   0,
    "Standard_L4s": 0,
    "Standard_L8s": 0,
    "Standard_L16s":    0,
    "Standard_L32s":    0,
    "Standard_M64s":    0,
    "Standard_M64ms":   0,
    "Standard_M128s":   0,
    "Standard_M128ms":  0,
    "Standard_M64-32ms":    0,
    "Standard_M64-16ms":    0,
    "Standard_M128-64ms":   0,
    "Standard_M128-32ms":   0,
    "Standard_NC6": 0,
    "Standard_NC12":    0,
    "Standard_NC24":    0,
    "Standard_NC24r":   0,
    "Standard_NC6s_v2": 0,
    "Standard_NC12s_v2":    0,
    "Standard_NC24s_v2":    0,
    "Standard_NC24rs_v2":   0,
    "Standard_NC6s_v3": 0,
    "Standard_NC12s_v3":    0,
    "Standard_NC24s_v3":    0,
    "Standard_NC24rs_v3":   0,
    "Standard_ND6s":    0,
    "Standard_ND12s":   0,
    "Standard_ND24s":   0,
    "Standard_ND24rs":  0,
    "Standard_NV6": 0,
    "Standard_NV12":    0,
    "Standard_NV24":    0
});
let CalculateCost = (value: dynamic) {
    // !! return real/int value !!
    CostMap[tostring(value)]
};
// **********************
// Step 3: (optional) update helper functions
let CompareCosts = (oldCost: real, newCost: real) {
    iff(isempty(newCost),
        -1 * oldCost,
        iff(isempty(newCost),
            newCost,
            newCost - oldCost))
};
let LabelCostChange = (costChange: real) {
    iff(costChange > 0,
        "Increase",
        iff(costChange < 0,
            "Decrease",
            "NoChange"))
};
// **********************
// Do not modify anything below.
MicrosoftChangeAnalysis_ChangeEvent_CL 
| where version_s =~ EventSchemaVersion and ResourceType =~ ResourceType and change_displayName_s =~ PropertyName
| project
    ChangeTime = change_timeStamp_t,
    ResourceId,
    change_oldValue = todynamic(change_oldValue_s),
    change_newValue = todynamic(change_newValue_s)
| project
    ChangeTime,
    ResourceId,
    OldCostValue = change_oldValue[ValueName],
    NewCostValue = change_newValue[ValueName]
| extend
    OldCost = CalculateCost(OldCostValue),
    NewCost = CalculateCost(NewCostValue)
| extend CostChange = CompareCosts(OldCost, NewCost)
| project
    ChangeTime,
    ResourceId,
    OldCostValue,
    NewCostValue,
    OldCost, // remove
    NewCost, // remove
    CostChange,
    CostChangeLabel = LabelCostChange(CostChange)
| order by ChangeTime asc

```