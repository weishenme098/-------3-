#include "HaikouMicrogridEMSAlgorithm.h"
#include <QDateTime>
#include <QDebug>
#include <QElapsedTimer>

HaikouMicrogridEMSAlgorithm::HaikouMicrogridEMSAlgorithm() {}

HaikouMicrogridEMSAlgorithm::~HaikouMicrogridEMSAlgorithm(){}

bool HaikouMicrogridEMSAlgorithm::HAIKOU_OVERALL_ALGO(const QJsonObject jsonInput, QJsonObject &jsonOutput)
{
    try
    {
        // 1. 解析JSON输入数据
        Configuration_Input config;
        ExternalPowerGrid_Input externalGrid;
        vector<Switch_Input> switches;
        ChangingDevice_Input changingDevice;
        vector<Integration_Input> integrationInputs;
        System_Output systemOutput;
        vector<Integration_Output> integrationOutputs;

        // 1.1 解析配置信息
        if (jsonInput.contains("Configuration"))
        {
            QJsonObject configObj = jsonInput["Configuration"].toObject();
            config.Flag_MicrogridType = configObj["Flag_MicrogridType"].toInt();
            config.Require_GridStatus = configObj["Require_GridStatus"].toInt();
            config.Un_Grid = configObj["Un_Grid"].toDouble();
            config.AllowableDeviation_Ugrid = configObj["AllowableDeviation_Ugrid"].toDouble();
            config.Un_Microgrid = configObj["Un_Microgrid"].toDouble();
            config.AllowableDeviation_Umicrogrid = configObj["AllowableDeviation_Umicrogrid"].toDouble();
            config.SOCmax_LiEss = configObj["SOCmax_LiEss"].toDouble();
            config.SOCmin_LiEss = configObj["SOCmin_LiEss"].toDouble();
            config.SOC_Autocharge = configObj["SOC_Autocharge"].toDouble();
            config.SOC_StopAutocharge = configObj["SOC_StopAutocharge"].toDouble();
            config.InitialSOC_EconomicDispatch = configObj["InitialSOC_EconomicDispatch"].toDouble();
            config.P_max_Load = configObj["P_max_Load"].toDouble();
            config.P_average_Load = configObj["P_average_Load"].toDouble();
            config.Time_OffgridOperation = configObj["Time_OffgridOperation"].toDouble();
            config.Time_Sunny = configObj["Time_Sunny"].toDouble();
            config.Time_Cloudy = configObj["Time_Cloudy"].toDouble();
            config.Time_Rainy = configObj["Time_Rainy"].toDouble();
            config.Maxnum_conwarnning = configObj["Maxnum_conwarnning"].toInt();
            config.Flag_WeatherStatus = configObj["Flag_WeatherStatus"].toInt();
            config.DeadZone_IncreaseActivePower = configObj["DeadZone_IncreaseActivePower"].toDouble();
            config.DeadZone_DecreaseActivePower = configObj["DeadZone_DecreaseActivePower"].toDouble();
            config.Kp_ActivePowerControl = configObj["Kp_ActivePowerControl"].toDouble();
            config.Ki_ActivePowerControl = configObj["Ki_ActivePowerControl"].toDouble();
            config.DeadZone_IncreaseReactivePower = configObj["DeadZone_IncreaseReactivePower"].toDouble();
            config.DeadZone_DecreaseReactivePower = configObj["DeadZone_DecreaseReactivePower"].toDouble();
            config.Kp_ReactivePowerControl = configObj["Kp_ReactivePowerControl"].toDouble();
            config.Ki_ReactivePowerControl = configObj["Ki_ReactivePowerControl"].toDouble();
            config.ControlCycle = configObj["ControlCycle"].toDouble();
            config.P_restrict_G2M = configObj["P_restrict_G2M"].toDouble();
            config.DeltaP_warn_G2M = configObj["DeltaP_warn_G2M"].toDouble();
            config.P_restrict_M2G = configObj["P_restrict_M2G"].toDouble();
            config.DeltaP_warn_M2G = configObj["DeltaP_warn_M2G"].toDouble();
            config.Qmax_pcc = configObj["Qmax_pcc"].toDouble();
            config.Qmin_pcc = configObj["Qmin_pcc"].toDouble();
            config.PF_max_pcc = configObj["PF_max_pcc"].toDouble();
            config.PF_min_pcc = configObj["PF_min_pcc"].toDouble();
            config.Uset_Grid = configObj["Uset_Grid"].toDouble();
            config.ControlFlag_OffgridFrequencyStability = configObj["ControlFlag_OffgridFrequencyStability"].toInt();
            config.ControlFlag_OffgridVoltageStability = configObj["ControlFlag_OffgridVoltageStability"].toInt();
            config.ControlFlag_GridConnectedActivePower = configObj["ControlFlag_GridConnectedActivePower"].toInt();
            config.ControlFlag_EconomicDispatch = configObj["ControlFlag_EconomicDispatch"].toInt();
            config.ControlFlag_GridConnectedReactivePower = configObj["ControlFlag_GridConnectedReactivePower"].toInt();
        }

        // 1.2 解析外部电网状态
        if (jsonInput.contains("ExternalPowerGrid"))
        {
            QJsonObject gridObj = jsonInput["ExternalPowerGrid"].toObject();
            externalGrid.F_real_pcc = gridObj["F_real_pcc"].toDouble();
            externalGrid.Ua_Grid = gridObj["Ua_Grid"].toDouble();
            externalGrid.Ub_Grid = gridObj["Ub_Grid"].toDouble();
            externalGrid.Uc_Grid = gridObj["Uc_Grid"].toDouble();
            externalGrid.Ua_Microgrid = gridObj["Ua_Microgrid"].toDouble();
            externalGrid.Ub_Microgrid = gridObj["Ub_Microgrid"].toDouble();
            externalGrid.Uc_Microgrid = gridObj["Uc_Microgrid"].toDouble();
            externalGrid.P_real_pcc = gridObj["P_real_pcc"].toDouble();
            externalGrid.P_previousone_pcc = gridObj["P_previousone_pcc"].toDouble();
            externalGrid.P_previoustwo_pcc = gridObj["P_previoustwo_pcc"].toDouble();
            externalGrid.P_previousthree_pcc = gridObj["P_previousthree_pcc"].toDouble();
            externalGrid.DeltaP_Microgrid_1 = gridObj["DeltaP_Microgrid_1"].toDouble();
            externalGrid.DeltaP_Microgrid_2 = gridObj["DeltaP_Microgrid_2"].toDouble();
            externalGrid.DeltaP_Microgrid_3 = gridObj["DeltaP_Microgrid_3"].toDouble();
            externalGrid.Pset_Microgrid_1 = gridObj["Pset_Microgrid_1"].toDouble();
            externalGrid.P0_max_1 = gridObj["P0_max_1"].toDouble();
            externalGrid.P0_min_1 = gridObj["P0_min_1"].toDouble();
            externalGrid.num1_conwarnning_1 = gridObj["num1_conwarnning_1"].toDouble();
            externalGrid.num2_conwarnning_1 = gridObj["num2_conwarnning_1"].toDouble();
            externalGrid.Flag_OverWarning_1 = gridObj["Flag_OverWarning_1"].toInt();
            externalGrid.TimeStatus = gridObj["TimeStatus"].toInt();
            externalGrid.WeatherStatus = gridObj["WeatherStatus"].toInt();
            externalGrid.Q_real_pcc = gridObj["Q_real_pcc"].toDouble();
            externalGrid.Q_previousone_pcc = gridObj["Q_previousone_pcc"].toDouble();
            externalGrid.Q_previoustwo_pcc = gridObj["Q_previoustwo_pcc"].toDouble();
            externalGrid.Q_previousthree_pcc = gridObj["Q_previousthree_pcc"].toDouble();
            externalGrid.DeltaQ_Microgrid_1 = gridObj["DeltaQ_Microgrid_1"].toDouble();
            externalGrid.DeltaQ_Microgrid_2 = gridObj["DeltaQ_Microgrid_2"].toDouble();
            externalGrid.DeltaQ_Microgrid_3 = gridObj["DeltaQ_Microgrid_3"].toDouble();
            externalGrid.Qset_Microgrid_1 = gridObj["Qset_Microgrid_1"].toDouble();
            externalGrid.PF_real_pcc = gridObj["PF_real_pcc"].toDouble();
        }

        // 1.3 解析开关状态
        if (jsonInput.contains("Switches"))
        {
            QJsonArray switchesArray = jsonInput["Switches"].toArray();
            for (const QJsonValue &switchVal : switchesArray)
            {
                QJsonObject switchObj = switchVal.toObject();
                Switch_Input switchItem;
                switchItem.ID = switchObj["ID"].toString().toStdString();
                switchItem.Type = switchObj["Type"].toInt();
                switchItem.FaultFlag = switchObj["FaultFlag"].toInt();
                switchItem.Status = switchObj["Status"].toInt();
                switches.push_back(switchItem);
            }
        }

        // 1.4 解析多路投切装置状态
        if (jsonInput.contains("ChangingDevice"))
        {
            QJsonObject deviceObj = jsonInput["ChangingDevice"].toObject();
            changingDevice.RunStatus = deviceObj["RunStatus"].toInt();
            changingDevice.CommunicationFaultStatus = deviceObj["CommunicationFaultStatus"].toInt();
            changingDevice.NonCommunicationFaultStatus = deviceObj["NonCommunicationFaultStatus"].toInt();
        }

        // 1.5 解析一体化设备输入
        if (jsonInput.contains("IntegrationInputs"))
        {
            QJsonArray integrationArray = jsonInput["IntegrationInputs"].toArray();
            for (const QJsonValue &integrationVal : integrationArray)
            {
                QJsonObject integrationObj = integrationVal.toObject();
                Integration_Input integrationItem;
                integrationItem.ID = integrationObj["ID"].toString().toStdString();
                //integrationItem.Key_Sort = integrationObj["Key_Sort"].toDouble();

                // 解析风电数据
                if (integrationObj.contains("Wind"))
                {
                    QJsonArray windArray = integrationObj["Wind"].toArray();
                    for (const QJsonValue &windVal : windArray)
                    {
                        QJsonObject windObj = windVal.toObject();
                        Wind_Input windItem;
                        windItem.ID_WT = windObj["ID_WT"].toString().toStdString();
                        windItem.Status = windObj["Status"].toInt();
                        windItem.Flag_Adjust = windObj["Flag_Adjust"].toInt();
                        windItem.P_real = windObj["P_real"].toDouble();
                        windItem.Pn = windObj["Pn"].toDouble();
                        windItem.P_max = windObj["P_max"].toDouble();
                        windItem.P_min = windObj["P_min"].toDouble();
                        integrationItem.Wind.push_back(windItem);
                    }
                }

                // 解析光伏DC/DC数据
                if (integrationObj.contains("DCDC_PV"))
                {
                    QJsonObject dcdcObj = integrationObj["DCDC_PV"].toObject();
                    DCDC_PV_Input dcdcItem;
                    dcdcItem.FaultStatus_DCDC = dcdcObj["FaultStatus_DCDC"].toInt();
                    dcdcItem.RunStatus_DCDC = dcdcObj["RunStatus_DCDC"].toInt();
                    dcdcItem.Flag_Adjust = dcdcObj["Flag_Adjust"].toInt();
                    dcdcItem.P_real = dcdcObj["P_real"].toDouble();
                    dcdcItem.Pn = dcdcObj["Pn"].toDouble();
                    dcdcItem.P_max = dcdcObj["P_max"].toDouble();
                    dcdcItem.P_min = dcdcObj["P_min"].toDouble();
                    dcdcItem.HighVoltage_real = dcdcObj["HighVoltage_real"].toDouble();
                    dcdcItem.I_max = dcdcObj["I_max"].toDouble();
                    integrationItem.DCDC_PV = dcdcItem;
                }

                // 解析储能数据
                if (integrationObj.contains("LiEss"))
                {
                    QJsonObject essObj = integrationObj["LiEss"].toObject();
                    LiEss_Input essItem;
                    essItem.Status = essObj["Status"].toInt();
                    essItem.RatedCapacity = essObj["RatedCapacity"].toDouble();
                    essItem.SOC_real = essObj["SOC_real"].toDouble();
                    essItem.P_real = essObj["P_real"].toDouble();
                    essItem.P_max = essObj["P_max"].toDouble();
                    essItem.P_min = essObj["P_min"].toDouble();
                    essItem.UnallowedChargingVoltage = essObj["UnallowedChargingVoltage"].toDouble();
                    essItem.TMSComState = essObj["TMSComState"].toInt();
                    essItem.ClusterMaxCellTemp = essObj["ClusterMaxCellTemp"].toDouble();
                    integrationItem.LiEss = essItem;
                }
                /*
                if (integrationObj.contains("Wind"))
                {
                    QJsonArray windArray = integrationObj["Wind"].toArray();
                    for (const QJsonValue& windVal : windArray)
                    {
                        QJsonObject windObj = windVal.toObject();
                        Wind_Input windItem;
                        windItem.ID_WT = windObj["ID_WT"].toString().toStdString();
                        windItem.Status = windObj["Status"].toInt();
                        windItem.Flag_Adjust = windObj["Flag_Adjust"].toInt();
                        windItem.P_real = windObj["P_real"].toDouble();
                        windItem.Pn = windObj["Pn"].toDouble();
                        windItem.P_max = windObj["P_max"].toDouble();
                        windItem.P_min = windObj["P_min"].toDouble();
                        integrationItem.Wind.push_back(windItem);
                    }
                }*/
                // 解析PCS数据
                if (integrationObj.contains("PCS"))
                {
                    QJsonArray pcsArray = integrationObj["PCS"].toArray();
                    for (const QJsonValue& pcsVal : pcsArray)
                    {
                        QJsonObject pcsObj = pcsVal.toObject();
                        PCS_Input pcsItem;
                        pcsItem.ID_PCS = pcsObj["ID_PCS"].toString().toStdString();
                        pcsItem.FaultStatus_PCS = pcsObj["FaultStatus_PCS"].toInt();
                        pcsItem.RunStatus_PCS = pcsObj["RunStatus_PCS"].toInt();
                        pcsItem.RunMode_PCS = pcsObj["RunMode_PCS"].toInt();
                        pcsItem.Type_PCS = pcsObj["Type_PCS"].toInt();
                        pcsItem.Flag_VFPCS = pcsObj["Flag_VFPCS"].toInt();
                        pcsItem.Pn = pcsObj["Pn"].toDouble();
                        pcsItem.P_real = pcsObj["P_real"].toDouble();
                        pcsItem.Q_real = pcsObj["Q_real"].toDouble();
                        pcsItem.P_max = pcsObj["P_max"].toDouble();
                        pcsItem.P_min = pcsObj["P_min"].toDouble();
                        pcsItem.P_set_1 = pcsObj["P_set_1"].toDouble();
                        pcsItem.Q_set_1 = pcsObj["Q_set_1"].toDouble();
                        integrationItem.PCS.push_back(pcsItem);
                    }
                }

                integrationInputs.push_back(integrationItem);
            }
        }

        // 初始化输出向量
        static bool firstCall = true;
        static bool previousCallFailed = false;
        if (firstCall || previousCallFailed)
        {
            systemOutput.Status_GridPCC = 1;  //对市电状态进行初始化
            systemOutput.Status_MicrogridBranch = 1;  //对微网支路状态进行初始化
            systemOutput.Flag_MicrogridSupply1 = 0;  // 初始化微网供电能力标志位为0（能力不足）
            systemOutput.Flag_MicrogridSupply2 = 0;
            systemOutput.DeltaP_Microgrid = 0.0;  //对微网总有功调节增量、总有功设定值初始化
            systemOutput.Pset_Microgrid = 0.0;
            systemOutput.Flag_OverWarning = 0;  // 当前周期未超预警，超限预警标志位置0
            /*
            systemOutput.P0_max = config.P_restrict_M2G - config.DeltaP_warn_M2G;  //上限控制参考值=上送预警值
            systemOutput.num1_conwarnning = 0;  //上限控制连续预警周期=0
            systemOutput.P0_min = config.P_restrict_G2M + config.DeltaP_warn_G2M;  //下限控制参考值=下送预警值
            systemOutput.num2_conwarnning = 0;  //下限控制连续预警周期=0
            */
            systemOutput.P0_max = externalGrid.P0_max_1;
            systemOutput.num1_conwarnning = externalGrid.num1_conwarnning_1;
            systemOutput.P0_min = externalGrid.P0_min_1;
            systemOutput.num2_conwarnning = externalGrid.num2_conwarnning_1;
            systemOutput.Flag_ExceedMicrogridCapability = 0;
            systemOutput.DeltaQ_Microgrid = 0.0;  //对微网总无功调节增量、总无功设定值初始化
            systemOutput.Qset_Microgrid = 0.0;
            systemOutput.Flag_ExceedQCapability = 0;

            integrationOutputs.resize(integrationInputs.size());
            for (size_t i = 0; i < integrationInputs.size(); ++i)
            {
                integrationOutputs[i].ID = integrationInputs[i].ID;
                integrationOutputs[i].Flag_Selected = 0;

                integrationOutputs[i].DCDC_PV.P_set = 0;
                integrationOutputs[i].DCDC_PV.HighVoltage_set = 0;
                integrationOutputs[i].DCDC_PV.Current_set = 0;
                integrationOutputs[i].DCDC_PV.StartStopControl_DCDC = 0;

                //integrationOutputs[i].wind.resize(integrationInputs[i].Wind.size());
                integrationOutputs[i].PCS.resize(integrationInputs[i].PCS.size());
                for (size_t j = 0; j < integrationOutputs[i].PCS.size(); ++j)
                {
                    integrationOutputs[i].PCS[j].ID_PCS = integrationInputs[i].PCS[j].ID_PCS;
                    integrationOutputs[i].PCS[j].FaultResetControl_PCS = 0;
                    integrationOutputs[i].PCS[j].Num_FaultReset = 0;
                    integrationOutputs[i].PCS[j].Flag_FaultResetFailure = 0;
                    integrationOutputs[i].PCS[j].StartStopControl_PCS = 0;
                    //integrationOutputs[i].PCS[j].RunModeControl_PCS = 0;
                    integrationOutputs[i].PCS[j].deltaP_set = 0.0;
                    integrationOutputs[i].PCS[j].P_set = 0.0;
                    integrationOutputs[i].PCS[j].deltaQ_set = 0.0;
                    integrationOutputs[i].PCS[j].Q_set = 0.0;

                    if (integrationInputs[i].PCS[j].Type_PCS == 1)
                    {
                        integrationOutputs[i].PCS[j].RunModeControl_PCS = 2;
                    }
                    else if (integrationInputs[i].PCS[j].Type_PCS == 2)
                    {
                        integrationOutputs[i].PCS[j].RunModeControl_PCS = 1;
                    }
                }
            }

            firstCall = false;  // 标记已经初始化过
            previousCallFailed = false;
        }
        else
        {
            systemOutput.Status_GridPCC = 1;  //对市电状态进行初始化
            systemOutput.Status_MicrogridBranch = 1;  //对微网支路状态进行初始化
            systemOutput.Flag_MicrogridSupply1 = 0;  // 初始化微网供电能力标志位为0（能力不足）
            systemOutput.Flag_MicrogridSupply2 = 0;
            systemOutput.DeltaP_Microgrid = 0.0;  //对微网总有功调节增量、总有功设定值初始化
            systemOutput.Pset_Microgrid = 0.0;
            systemOutput.Flag_OverWarning = 0;  // 当前周期未超预警，超限预警标志位置0
            systemOutput.P0_max = externalGrid.P0_max_1;
            systemOutput.num1_conwarnning = externalGrid.num1_conwarnning_1;
            systemOutput.P0_min = externalGrid.P0_min_1;
            systemOutput.num2_conwarnning = externalGrid.num2_conwarnning_1;
            systemOutput.Flag_ExceedMicrogridCapability = 0;
            systemOutput.DeltaQ_Microgrid = 0.0;  //对微网总无功调节增量、总无功设定值初始化
            systemOutput.Qset_Microgrid = 0.0;
            systemOutput.Flag_ExceedQCapability = 0;

            if(integrationOutputs.size() != integrationInputs.size())
            {
                integrationOutputs.resize(integrationInputs.size());
            }

            for (size_t i = 0; i < integrationInputs.size(); ++i)
            {
                integrationOutputs[i].ID = integrationInputs[i].ID;
                integrationOutputs[i].Flag_Selected = 0;

                integrationOutputs[i].DCDC_PV.P_set = 0;
                integrationOutputs[i].DCDC_PV.HighVoltage_set = 0;
                integrationOutputs[i].DCDC_PV.Current_set = 0;
                integrationOutputs[i].DCDC_PV.StartStopControl_DCDC = 0;


                if(integrationOutputs[i].PCS.size() != integrationInputs[i].PCS.size())
                {
                    integrationOutputs[i].PCS.resize(integrationInputs[i].PCS.size());
                }

                for (size_t j = 0; j < integrationInputs[i].PCS.size(); ++j)
                {
                    integrationOutputs[i].PCS[j].ID_PCS = integrationInputs[i].PCS[j].ID_PCS;
                    integrationOutputs[i].PCS[j].FaultResetControl_PCS = 0;
                    integrationOutputs[i].PCS[j].Num_FaultReset = 0;
                    integrationOutputs[i].PCS[j].Flag_FaultResetFailure = 0;
                    integrationOutputs[i].PCS[j].StartStopControl_PCS = 0;
                    integrationOutputs[i].PCS[j].deltaP_set = 0.0;
                    integrationOutputs[i].PCS[j].P_set = 0.0;
                    integrationOutputs[i].PCS[j].deltaQ_set = 0.0;
                    integrationOutputs[i].PCS[j].Q_set = 0.0;

                    if (integrationInputs[i].PCS[j].Type_PCS == 1)
                    {
                        integrationOutputs[i].PCS[j].RunModeControl_PCS = 2;
                    }
                    else if (integrationInputs[i].PCS[j].Type_PCS == 2)
                    {
                        integrationOutputs[i].PCS[j].RunModeControl_PCS = 1;
                    }
                }
            }
        }

        // 2. 调用核心算法
        bool success = OverallFunction(config, externalGrid, switches, changingDevice, integrationInputs, systemOutput, integrationOutputs);
        firstCall = false;  // 标记已经初始化过
        previousCallFailed = false;  //调用成功，失败标志置0

        if (!success)
        {
            jsonOutput["status"] = "error";
            jsonOutput["message"] = "Algorithm execution failed";
            previousCallFailed = true;   //调用失败，失败标志置1
            return false;
        }

        // 3. 生成JSON输出
        // 3.1 系统输出
        QJsonObject systemOutputObj;
        systemOutputObj["Status_GridPCC"] = systemOutput.Status_GridPCC;
        systemOutputObj["Status_MicrogridBranch"] = systemOutput.Status_MicrogridBranch;
        systemOutputObj["Flag_MicrogridSupply1"] = systemOutput.Flag_MicrogridSupply1;
        systemOutputObj["Flag_MicrogridSupply2"] = systemOutput.Flag_MicrogridSupply2;
        systemOutputObj["DeltaP_Microgrid"] = systemOutput.DeltaP_Microgrid;
        systemOutputObj["Pset_Microgrid"] = systemOutput.Pset_Microgrid;
        systemOutputObj["P0_max"] = systemOutput.P0_max;
        systemOutputObj["P0_min"] = systemOutput.P0_min;
        systemOutputObj["num1_conwarnning"] = systemOutput.num1_conwarnning;
        systemOutputObj["num2_conwarnning"] = systemOutput.num2_conwarnning;
        systemOutputObj["Flag_OverWarning"] = systemOutput.Flag_OverWarning;
        systemOutputObj["Flag_ExceedMicrogridCapability"] = systemOutput.Flag_ExceedMicrogridCapability;
        systemOutputObj["DeltaQ_Microgrid"] = systemOutput.DeltaQ_Microgrid;
        systemOutputObj["Qset_Microgrid"] = systemOutput.Qset_Microgrid;
        systemOutputObj["Flag_ExceedQCapability"] = systemOutput.Flag_ExceedQCapability;
        jsonOutput["SystemOutput"] = systemOutputObj;

        // 3.2 一体化设备输出
        QJsonArray integrationOutputsArray;
        for (size_t i = 0; i < integrationOutputs.size(); ++i)
        {
            QJsonObject integrationOutputObj;
            integrationOutputObj["ID"] = QString::fromStdString(integrationOutputs[i].ID);
            integrationOutputObj["Flag_Selected"] = integrationOutputs[i].Flag_Selected;
            /*
            // 风电输出
            QJsonArray windOutputArray;
            for (size_t j = 0; j < integrationOutputs[i].wind.size(); ++j)
            {
                QJsonObject windOutputObj;
                windOutputObj["ID_WT"] = QString::fromStdString(integrationOutputs[i].wind[j].ID_WT);
                windOutputObj["P_set"] = integrationOutputs[i].wind[j].P_set;
                windOutputArray.append(windOutputObj);
            }
            integrationOutputObj["Wind"] = windOutputArray;
            */
            // DC/DC光伏输出
            QJsonObject dcdcOutputObj;
            dcdcOutputObj["P_set"] = integrationOutputs[i].DCDC_PV.P_set;
            dcdcOutputObj["HighVoltage_set"] = integrationOutputs[i].DCDC_PV.HighVoltage_set;
            dcdcOutputObj["Current_set"] = integrationOutputs[i].DCDC_PV.Current_set;
            dcdcOutputObj["StartStopControl_DCDC"] = integrationOutputs[i].DCDC_PV.StartStopControl_DCDC;
            integrationOutputObj["DCDC_PV"] = dcdcOutputObj;

            // PCS输出
            QJsonArray pcsOutputArray;
            for (size_t j = 0; j < integrationOutputs[i].PCS.size(); ++j)
            {
                QJsonObject pcsOutputObj;
                pcsOutputObj["ID_PCS"] = QString::fromStdString(integrationOutputs[i].PCS[j].ID_PCS);
                pcsOutputObj["FaultResetControl_PCS"] = integrationOutputs[i].PCS[j].FaultResetControl_PCS;
                pcsOutputObj["Num_FaultReset"] = integrationOutputs[i].PCS[j].Num_FaultReset;
                pcsOutputObj["Flag_FaultResetFailure"] = integrationOutputs[i].PCS[j].Flag_FaultResetFailure;
                pcsOutputObj["StartStopControl_PCS"] = integrationOutputs[i].PCS[j].StartStopControl_PCS;
                pcsOutputObj["RunModeControl_PCS"] = integrationOutputs[i].PCS[j].RunModeControl_PCS;
                pcsOutputObj["deltaP_set"] = integrationOutputs[i].PCS[j].deltaP_set;
                pcsOutputObj["P_set"] = integrationOutputs[i].PCS[j].P_set;
                pcsOutputObj["deltaQ_set"] = integrationOutputs[i].PCS[j].deltaQ_set;
                pcsOutputObj["Q_set"] = integrationOutputs[i].PCS[j].Q_set;
                pcsOutputArray.append(pcsOutputObj);
            }
            integrationOutputObj["PCS"] = pcsOutputArray;

            integrationOutputsArray.append(integrationOutputObj);
        }
        jsonOutput["IntegrationOutputs"] = integrationOutputsArray;

        jsonOutput["status"] = "success";
        jsonOutput["message"] = "Algorithm executed successfully";
        return true;
    }
    catch (const std::exception &e)
    {
        // 异常发生，设置标志以便下次重新初始化
        static bool firstCall = true;
        static bool previousCallFailed = false;
        previousCallFailed = true;
        
        jsonOutput["status"] = "error";
        jsonOutput["message"] = QString("Exception occurred: %1").arg(e.what());
        return false;
    }
    catch (...)
    {
        // 未知异常发生，设置标志以便下次重新初始化
        static bool firstCall = true;
        static bool previousCallFailed = false;
        previousCallFailed = true;

        jsonOutput["status"] = "error";
        jsonOutput["message"] = "Unknown exception occurred";
        return false;
    }
}

bool HaikouMicrogridEMSAlgorithm::OverallFunction(Configuration_Input Configuration, ExternalPowerGrid_Input ExternalPowerGrid, vector<Switch_Input> Switch, ChangingDevice_Input ChangingDevice, vector<Integration_Input> IntegrationInput, System_Output& SystemOutput, vector<Integration_Output>& IntegrationOutput)
{
    //纯离网型微电网
    if (Configuration.Flag_MicrogridType == 1)
    {
        //判断是否符合微网离网运行条件:“市电三相电压任意一相不正常且市电开关断开、微网接入点三相电压任意一相不正常且微网内其他电压源型电源开关断开、微网支路开关均闭合”三个条件同时满足
        if ((fabs(ExternalPowerGrid.Ua_Grid - Configuration.Un_Grid) > Configuration.AllowableDeviation_Ugrid) || (fabs(ExternalPowerGrid.Ub_Grid - Configuration.Un_Grid) > Configuration.AllowableDeviation_Ugrid) || (fabs(ExternalPowerGrid.Uc_Grid - Configuration.Un_Grid) > Configuration.AllowableDeviation_Ugrid))  //市电三相电压不正常
        {
            for (size_t i = 0;i < Switch.size();i = i + 1)
            {
                if (Switch[i].Type == 0)  //市电开关
                {
                    if (Switch[i].Status == 0)  //有任意一个市电开关断开
                    {
                        break;  //跳出for循环本身，执行for循环后的语句
                    }
                }
                else
                {
                    continue;  //不是市电开关，跳出当前迭代，开始下一次迭代
                }
            }
            if ((fabs(ExternalPowerGrid.Ua_Microgrid - Configuration.Un_Microgrid) > Configuration.AllowableDeviation_Umicrogrid) || (fabs(ExternalPowerGrid.Ub_Microgrid - Configuration.Un_Microgrid) > Configuration.AllowableDeviation_Umicrogrid) || (fabs(ExternalPowerGrid.Uc_Microgrid - Configuration.Un_Microgrid) > Configuration.AllowableDeviation_Umicrogrid))  //微网接入点三相电压不正常
            {
                for (size_t j = 0;j < Switch.size();j = j + 1)
                {
                    if (Switch[j].Type == 2)  //如果是其他电压源型电源的开关
                    {
                        if (Switch[j].Status == 0)
                        {
                            continue;  //其他电压源型电源开关断开，跳出当前迭代，开始下一次迭代
                        }
                        else
                        {
                            return true;  //其他电压源型电源开关闭合，不满足微网离网运行条件
                        }
                    }
                    else if (Switch[j].Type == 1)  //如果是微网支路开关
                    {
                        if (Switch[j].Status == 0)
                        {
                            return true;  //微网支路开关断开，不满足微网离网运行条件
                        }
                        else
                        {
                            continue;  //微网支路开关闭合，跳出当前迭代，开始下一次迭代
                        }
                    }
                    else
                    {
                        continue;  //既不是电压源型电源的开关，又不是微网支路开关，跳出当前迭代，开始下一次迭代
                    }
                }

                //控制需要工作于VF模式的PCS进入VF模块
                PCSVFModeControlFunction(IntegrationInput, IntegrationOutput);

                //进入微电网离网运行控制
                if (Configuration.ControlFlag_OffgridFrequencyStability == 1)
                {
                    OffgridOperationFrequencyControlFunction();  //离网运行频率控制函数。！！！该函数还未开发，待完善。
                }
                if (Configuration.ControlFlag_OffgridVoltageStability == 1)
                {
                    OffgridOperationVoltageControlFunction();  //离网运行电压控制函数。！！！该函数还未开发，待完善。
                }
            }
            else
            {
                return true;  //微网接入点电压正常，不满足微网离网运行条件
            }
        }
        else
        {
            return true;  //市电电压正常，不满足微网离网运行条件
        }
    }

    //纯并网型微电网
    else if (Configuration.Flag_MicrogridType == 2)
    {
        ExternalPowerGridStatusJudgmentFunction(Configuration, ExternalPowerGrid, Switch, SystemOutput);  //外部电网状态判断函数，判断是否符合微电网并网运行条件

        //微网支路不正常，控制所有PCS停机
        if (SystemOutput.Status_MicrogridBranch == 0)
        {
            for (size_t i = 0; i < IntegrationInput.size(); i = i + 1)
            {
                for (size_t j = 0; j < IntegrationInput[i].PCS.size(); j = j + 1)
                {
                    if (IntegrationInput[i].PCS[j].Type_PCS == 1)
                    {
                        if (IntegrationInput[i].PCS[j].RunStatus_PCS != 0)  //该PCS当前未处于停机状态
                        {
                            IntegrationOutput[i].PCS[j].ID_PCS = IntegrationInput[i].PCS[j].ID_PCS;
                            IntegrationOutput[i].PCS[j].deltaP_set = -IntegrationInput[i].PCS[j].P_real;
                            IntegrationOutput[i].PCS[j].P_set = 0;
                            IntegrationOutput[i].PCS[j].StartStopControl_PCS = 0;  //输出：控制该PCS停机
                        }
                    }
                    else
                    {
                        continue;
                    }
                }
            }
        }
        //微网支路正常，进一步根据市电状态进行分别控制
        else if (SystemOutput.Status_MicrogridBranch == 1)
        {
            //一、市电正常且微网支路正常时：控制所有PCS工作于PQ模式，并进行微电网并网有功功率控制和微电网并网无功功率控制。
            if (SystemOutput.Status_GridPCC == 1)
            {
                //控制所有PCS处于PQ模式
                PCSPQModeControlFunction(IntegrationInput, IntegrationOutput);

                for (size_t i = 0; i < IntegrationInput.size(); i = i + 1)
                {
                    for (size_t j = 0; j < IntegrationInput[i].PCS.size(); j = j + 1)
                    {
                        if (IntegrationInput[i].PCS[j].Type_PCS == 1)
                        {
                            if (IntegrationInput[i].PCS[j].RunMode_PCS == 1)
                            {
                                return true;  //有PCS仍运行于VF模式，微网不能并网运行
                            }
                            else
                            {
                                continue;
                            }
                        }
                        else
                        {
                            continue;
                        }
                    }
                }

                if (Configuration.ControlFlag_GridConnectedActivePower == 1)
                {
                    //微网并网有功功率控制函数
                    GridConnectedActivePowerControlFunction(Configuration, ExternalPowerGrid, IntegrationInput, SystemOutput, IntegrationOutput);
                }
                if (Configuration.ControlFlag_GridConnectedReactivePower == 1)
                {
                    //微网并网无功功率控制函数
                    GridConnectedReactivePowerControlFunction(Configuration, ExternalPowerGrid, IntegrationInput, SystemOutput, IntegrationOutput);
                }
            }
            //二、市电不正常，但微网支路正常时：分两种情况分别控制。
            else if (SystemOutput.Status_GridPCC == 0)
            {
                //情况1：外部有其他电压源型电源(如柴发)作主电源，微电网仍并网运行，发出功率以补充超出柴发最大发电能力的部分，或者在柴发最大发电能力范围内利用柴发多余电力给储能充电。
                if (Configuration.Require_GridStatus == 0)
                {
                    PCSPQModeControlFunction(IntegrationInput, IntegrationOutput);  //控制所有PCS处于PQ模式
                    for (size_t i = 0; i < IntegrationInput.size(); i = i + 1)
                    {
                        for (size_t j = 0; j < IntegrationInput[i].PCS.size(); j = j + 1)
                        {
                            if (IntegrationInput[i].PCS[j].Type_PCS == 1)
                            {
                                if (IntegrationInput[i].PCS[j].RunMode_PCS == 1)
                                {
                                    return true;  //有PCS仍运行于VF模式，微网不能并网运行
                                }
                                else
                                {
                                    continue;
                                }
                            }
                            else
                            {
                                continue;
                            }
                        }
                    }
                    if (Configuration.ControlFlag_GridConnectedActivePower == 1)
                    {
                        //发出功率以补充超出柴发最大发电能力的部分，或者在柴发最大发电能力范围内利用柴发多余电力给储能充电
                        //1）、先确定微电网总有功功率设定值和总有功目标调节增量
                        SystemOutput.Pset_Microgrid = -30;  //！！！后续需优化为配置量
                        double Preal_Microgrid = 0;  //微电网实际功率
                        for (size_t i = 0; i < IntegrationInput.size(); i = i + 1)
                        {
                            for (size_t j = 0; j < IntegrationInput[i].PCS.size(); j = j + 1)
                            {
                                if (IntegrationInput[i].PCS[j].Type_PCS == 1)
                                {
                                    Preal_Microgrid = Preal_Microgrid + IntegrationInput[i].PCS[j].P_real;
                                }
                            }
                        }
                        SystemOutput.DeltaP_Microgrid = SystemOutput.Pset_Microgrid - Preal_Microgrid;  //输出：微网总有功目标调节增量

                        //2）、进行增量分配，分为三种情况：
                        if (SystemOutput.DeltaP_Microgrid > Configuration.DeadZone_IncreaseActivePower)	//微网总有功目标调节增量>升功率控制死区时，进行微网有功升功率控制
                        {
                            //微网升有功功率分配控制与有功设定值计算函数
                            IncreaseActivePowerAllocateControlFunction(Configuration, IntegrationInput, SystemOutput, IntegrationOutput);
                        }
                        else if (SystemOutput.DeltaP_Microgrid < Configuration.DeadZone_DecreaseActivePower)  //微网总有功目标调节增量<降功率控制死区时，进行微网有功降功率控制
                        {
                            //微网降有功功率分配控制与有功设定值计算函数
                            DecreaseActivePowerAllocateControlFunction(Configuration, IntegrationInput, SystemOutput, IntegrationOutput);
                        }
                        else if ((SystemOutput.DeltaP_Microgrid <= Configuration.DeadZone_IncreaseActivePower) && (SystemOutput.DeltaP_Microgrid >= Configuration.DeadZone_DecreaseActivePower))  //在升降功率控制死区范围内，微网有功不增不减，维持不变
                        {
                            SystemOutput.DeltaP_Microgrid = 0;
                            for (size_t i = 0; i < IntegrationInput.size(); i = i + 1)
                            {
                                for (size_t j = 0; j < IntegrationInput[i].PCS.size(); j = j + 1)
                                {
                                    if (IntegrationInput[i].PCS[j].Type_PCS == 1)
                                    {
                                        IntegrationOutput[i].PCS[j].ID_PCS = IntegrationInput[i].PCS[j].ID_PCS;
                                        IntegrationOutput[i].PCS[j].deltaP_set = 0;
                                        PCSActivePowerSetCalculateFunction(IntegrationInput[i].PCS[j], IntegrationOutput[i].PCS[j]);  //第i个风光储单元有功设定值的计算与修正，更新IntegrationOutput[i].PCS[j].P_set
                                        SystemOutput.Pset_Microgrid = SystemOutput.Pset_Microgrid + IntegrationOutput[i].PCS[j].P_set;
                                    }
                                }
                                
                            }
                        }
                    }
                }
                //情况2：只有市电作主电源微电网才并网运行，即要求市电状态与微网支路都正常，否则微网不对外发出或吸收功率
                else if (Configuration.Require_GridStatus == 1)
                {
                    //微网不对外发出或吸收功率，控制所有PCS停机
                    for (size_t i = 0; i < IntegrationInput.size(); i = i + 1)
                    {
                        for (size_t j = 0; j < IntegrationInput[i].PCS.size(); j = j + 1)
                        {
                            if (IntegrationInput[i].PCS[j].Type_PCS == 1)
                            {
                                if (IntegrationInput[i].PCS[j].RunStatus_PCS != 0)  //该PCS当前未处于停机状态
                                {
                                    IntegrationOutput[i].PCS[j].ID_PCS = IntegrationInput[i].PCS[j].ID_PCS;
                                    IntegrationOutput[i].PCS[j].deltaP_set = -IntegrationInput[i].PCS[j].P_real;
                                    IntegrationOutput[i].PCS[j].P_set = 0;
                                    IntegrationOutput[i].PCS[j].StartStopControl_PCS = 0;  //输出：控制该PCS停机
                                }
                            }
                            else
                            {
                                continue;
                            }
                        }
                    }
                }
            }
        }
    }

    //并离网型微电网
    else if (Configuration.Flag_MicrogridType == 3)
    {
        //多路投切装置当前处于未投运状态，按照纯并网型微网运行控制
        if (ChangingDevice.RunStatus == 0)
        {
            ExternalPowerGridStatusJudgmentFunction(Configuration, ExternalPowerGrid, Switch, SystemOutput);  //外部电网状态判断函数，判断是否符合微电网并网运行条件

            //微网支路不正常，控制所有PCS停机
            if (SystemOutput.Status_MicrogridBranch == 0)
            {
                for (size_t i = 0; i < IntegrationInput.size(); i = i + 1)
                {
                    for (size_t j = 0; j < IntegrationInput[i].PCS.size(); j = j + 1)
                    {
                        if (IntegrationInput[i].PCS[j].Type_PCS == 1)
                        {
                            if (IntegrationInput[i].PCS[j].RunStatus_PCS != 0)  //该PCS当前未处于停机状态
                            {
                                IntegrationOutput[i].PCS[j].ID_PCS = IntegrationInput[i].PCS[j].ID_PCS;
                                IntegrationOutput[i].PCS[j].deltaP_set = -IntegrationInput[i].PCS[j].P_real;
                                IntegrationOutput[i].PCS[j].P_set = 0;
                                IntegrationOutput[i].PCS[j].StartStopControl_PCS = 0;  //输出：控制该PCS停机
                            }
                        }
                        else
                        {
                            continue;
                        }
                    }
                }
            }
            //微网支路正常，进一步根据市电状态进行分别控制
            else if (SystemOutput.Status_MicrogridBranch == 1)
            {
                //一、市电正常且微网支路正常时：控制所有PCS工作于PQ模式，并进行微电网并网有功功率控制和微电网并网无功功率控制。
                if (SystemOutput.Status_GridPCC == 1)
                {
                    //控制所有PCS处于PQ模式
                    PCSPQModeControlFunction(IntegrationInput, IntegrationOutput);

                    for (size_t i = 0; i < IntegrationInput.size(); i = i + 1)
                    {
                        for (size_t j = 0; j < IntegrationInput[i].PCS.size(); j = j + 1)
                        {
                            if (IntegrationInput[i].PCS[j].Type_PCS == 1)
                            {
                                if ((IntegrationInput[i].PCS[j].RunMode_PCS == 1) && (IntegrationInput[i].PCS[j].RunStatus_PCS == 1))
                                {
                                    IntegrationOutput[i].PCS[j].ID_PCS = IntegrationInput[i].PCS[j].ID_PCS;
                                    IntegrationOutput[i].PCS[j].StartStopControl_PCS = 0;  //输出：控制该PCS停机
                                    IntegrationOutput[i].PCS[j].P_set = 0;
                                    //return true;  //有PCS仍运行于VF模式，微网不能并网运行
                                }
                                else
                                {
                                    continue;
                                }
                            }
                            else
                            {
                                continue;
                            }
                        }
                    }

                    if (Configuration.ControlFlag_GridConnectedActivePower == 1)
                    {
                        //微网并网有功功率控制函数
                        GridConnectedActivePowerControlFunction(Configuration, ExternalPowerGrid, IntegrationInput, SystemOutput, IntegrationOutput);
                    }
                    if (Configuration.ControlFlag_GridConnectedReactivePower == 1)
                    {
                        //微网并网无功功率控制函数
                        GridConnectedReactivePowerControlFunction(Configuration, ExternalPowerGrid, IntegrationInput, SystemOutput, IntegrationOutput);
                    }
                }
                //二、市电不正常，但微网支路正常时：分两种情况分别控制。
                else if (SystemOutput.Status_GridPCC == 0)
                {
                    //情况1：外部有其他电压源型电源(如柴发)作主电源，微电网仍并网运行，发出功率以补充超出柴发最大发电能力的部分，或者在柴发最大发电能力范围内利用柴发多余电力给储能充电。
                    if (Configuration.Require_GridStatus == 0)
                    {
                        PCSPQModeControlFunction(IntegrationInput, IntegrationOutput);  //控制所有PCS处于PQ模式
                        for (size_t i = 0; i < IntegrationInput.size(); i = i + 1)
                        {
                            for (size_t j = 0; j < IntegrationInput[i].PCS.size(); j = j + 1)
                            {
                                if (IntegrationInput[i].PCS[j].Type_PCS == 1)
                                {
                                    if ((IntegrationInput[i].PCS[j].RunMode_PCS == 1)&&(IntegrationInput[i].PCS[j].RunStatus_PCS==1))
                                    {
                                        IntegrationOutput[i].PCS[j].ID_PCS = IntegrationInput[i].PCS[j].ID_PCS;
                                        IntegrationOutput[i].PCS[j].StartStopControl_PCS = 0;  //输出：控制该PCS停机
                                        IntegrationOutput[i].PCS[j].P_set = 0;
                                        //return true;  //有PCS仍运行于VF模式，微网不能并网运行
                                    }
                                    else
                                    {
                                        continue;
                                    }
                                }
                                else
                                {
                                    continue;
                                }
                            }
                        }
                        if (Configuration.ControlFlag_GridConnectedActivePower == 1)
                        {
                            //发出功率以补充超出柴发最大发电能力的部分，或者在柴发最大发电能力范围内利用柴发多余电力给储能充电
                            //1）、先确定微电网总有功功率设定值和总有功目标调节增量
                            SystemOutput.Pset_Microgrid = -20;  //！！！后续需优化为配置量
                            double Preal_Microgrid = 0;  //微电网实际功率
                            for (size_t i = 0; i < IntegrationInput.size(); i = i + 1)
                            {
                                for (size_t j = 0; j < IntegrationInput[i].PCS.size(); j = j + 1)
                                {
                                    if (IntegrationInput[i].PCS[j].Type_PCS == 1)
                                    {
                                        Preal_Microgrid = Preal_Microgrid + IntegrationInput[i].PCS[j].P_real;
                                    }
                                }
                            }
                            SystemOutput.DeltaP_Microgrid = SystemOutput.Pset_Microgrid - Preal_Microgrid;  //输出：微网总有功目标调节增量

                            //2）、进行增量分配，分为三种情况：
                            if (SystemOutput.DeltaP_Microgrid > Configuration.DeadZone_IncreaseActivePower)	//微网总有功目标调节增量>升功率控制死区时，进行微网有功升功率控制
                            {
                                //微网升有功功率分配控制与有功设定值计算函数
                                IncreaseActivePowerAllocateControlFunction(Configuration, IntegrationInput, SystemOutput, IntegrationOutput);
                            }
                            else if (SystemOutput.DeltaP_Microgrid < Configuration.DeadZone_DecreaseActivePower)  //微网总有功目标调节增量<降功率控制死区时，进行微网有功降功率控制
                            {
                                //微网降有功功率分配控制与有功设定值计算函数
                                DecreaseActivePowerAllocateControlFunction(Configuration, IntegrationInput, SystemOutput, IntegrationOutput);
                            }
                            else if ((SystemOutput.DeltaP_Microgrid <= Configuration.DeadZone_IncreaseActivePower) && (SystemOutput.DeltaP_Microgrid >= Configuration.DeadZone_DecreaseActivePower))  //在升降功率控制死区范围内，微网有功不增不减，维持不变
                            {
                                SystemOutput.DeltaP_Microgrid = 0;
                                for (size_t i = 0; i < IntegrationInput.size(); i = i + 1)
                                {
                                    for (size_t j = 0; j < IntegrationInput[i].PCS.size(); j = j + 1)
                                    {
                                        if (IntegrationInput[i].PCS[j].Type_PCS == 1)
                                        {
                                            IntegrationOutput[i].PCS[j].ID_PCS = IntegrationInput[i].PCS[j].ID_PCS;
                                            IntegrationOutput[i].PCS[j].deltaP_set = 0;
                                            PCSActivePowerSetCalculateFunction(IntegrationInput[i].PCS[j], IntegrationOutput[i].PCS[j]);  //第i个风光储单元有功设定值的计算与修正，更新IntegrationOutput[i].PCS[j].P_set
                                            SystemOutput.Pset_Microgrid = SystemOutput.Pset_Microgrid + IntegrationOutput[i].PCS[j].P_set;
                                        }
                                    }
                                }
                            }
                        }
                    }
                    //情况2：只有市电作主电源微电网才并网运行，即要求市电状态与微网支路都正常，否则微网不对外发出或吸收功率
                    else if (Configuration.Require_GridStatus == 1)
                    {
                        //微网不对外发出或吸收功率，控制所有PCS停机
                        for (size_t i = 0; i < IntegrationInput.size(); i = i + 1)
                        {
                            for (size_t j = 0; j < IntegrationInput[i].PCS.size(); j = j + 1)
                            {
                                if (IntegrationInput[i].PCS[j].Type_PCS == 1)
                                {
                                    if (IntegrationInput[i].PCS[j].RunStatus_PCS != 0)  //该PCS当前未处于停机状态
                                    {
                                        IntegrationOutput[i].PCS[j].ID_PCS = IntegrationInput[i].PCS[j].ID_PCS;
                                        IntegrationOutput[i].PCS[j].deltaP_set = -IntegrationInput[i].PCS[j].P_real;
                                        IntegrationOutput[i].PCS[j].P_set = 0;
                                        IntegrationOutput[i].PCS[j].StartStopControl_PCS = 0;  //输出：控制该PCS停机
                                    }
                                }
                                else
                                {
                                    continue;
                                }
                            }
                        }
                    }
                }
            }
        }
        //多路投切装置当前处于投运状态，按照并离网型微网运行控制
        else if (ChangingDevice.RunStatus == 1)
        {
            MicrogridPowerSupplyCapacityJudgmentFunction(Configuration, IntegrationInput, SystemOutput);  //微网供电能力判断

            for (size_t i = 0;i < IntegrationInput.size();i = i + 1)
            {
                for (size_t j = 0; j < IntegrationInput[i].PCS.size(); j = j + 1)
                {
                    if (IntegrationInput[i].PCS[j].Type_PCS == 1 && IntegrationInput[i].PCS[j].Flag_VFPCS == 1)
                    {
                        //作为主电源的PCS已处于VF模式，即代表微网正在离网运行
                        if (IntegrationInput[i].PCS[j].RunMode_PCS == 1)
                        {
                            if (IntegrationInput[i].PCS[j].RunStatus_PCS ==1)
                            {
                                //进入微电网离网运行控制
                                if (Configuration.ControlFlag_OffgridFrequencyStability == 1)
                                {
                                    OffgridOperationFrequencyControlFunction();  //离网运行频率控制函数。！！！该函数还未开发，待完善。
                                }
                                if (Configuration.ControlFlag_OffgridVoltageStability == 1)
                                {
                                    OffgridOperationVoltageControlFunction();  //离网运行电压控制函数。！！！该函数还未开发，待完善。
                                }
                            }
                        }
                        //作为主电源的PCS已处于PQ模式，即代表微网可根据外部电网状态并网运行
                        else if (IntegrationInput[i].PCS[j].RunMode_PCS == 2)
                        {
                            ExternalPowerGridStatusJudgmentFunction(Configuration, ExternalPowerGrid, Switch, SystemOutput);  //外部电网状态判断函数，判断是否符合微电网并网运行条件

                            //微网支路不正常，所有未停机PCS的有功功率目标值设为0
                            if (SystemOutput.Status_MicrogridBranch == 0)
                            {
                                for (size_t i = 0; i < IntegrationInput.size(); i = i + 1)
                                {
                                    for (size_t j = 0; j < IntegrationInput[i].PCS.size(); j = j + 1)
                                    {
                                        if (IntegrationInput[i].PCS[j].Type_PCS == 1)
                                        {
                                            if (IntegrationInput[i].PCS[j].RunStatus_PCS != 0)  //该PCS当前未处于停机状态
                                            {
                                                IntegrationOutput[i].PCS[j].ID_PCS = IntegrationInput[i].PCS[j].ID_PCS;
                                                IntegrationOutput[i].PCS[j].deltaP_set = -IntegrationInput[i].PCS[j].P_real;
                                                IntegrationOutput[i].PCS[j].P_set = 0;
                                            }
                                        }
                                        else
                                        {
                                            continue;
                                        }
                                    }
                                }
                            }
                            //只有在微网支路正常的情况下，再进一步根据市电状态进行分别控制
                            else if (SystemOutput.Status_MicrogridBranch == 1)
                            {
                                //一、市电正常且微网支路正常时：控制所有PCS工作于PQ模式，并进行微电网并网有功功率控制和微电网并网无功功率控制。
                                if (SystemOutput.Status_GridPCC == 1)
                                {
                                    if (Configuration.ControlFlag_GridConnectedActivePower == 1)
                                    {
                                        //微网并网有功功率控制函数
                                        GridConnectedActivePowerControlFunction(Configuration, ExternalPowerGrid, IntegrationInput, SystemOutput, IntegrationOutput);
                                    }
                                    if (Configuration.ControlFlag_GridConnectedReactivePower == 1)
                                    {
                                        //微网并网无功功率控制函数
                                        GridConnectedReactivePowerControlFunction(Configuration, ExternalPowerGrid, IntegrationInput, SystemOutput, IntegrationOutput);
                                    }
                                }
                                //二、市电不正常，但微网支路正常时：分两种情况分别控制。
                                else if (SystemOutput.Status_GridPCC == 0)
                                {
                                    //情况1：外部有其他电压源型电源(如柴发)作主电源，微电网仍并网运行，发出功率以补充超出柴发最大发电能力的部分，或者在柴发最大发电能力范围内利用柴发多余电力给储能充电。
                                    if (Configuration.Require_GridStatus == 0)
                                    {
                                        if (Configuration.ControlFlag_GridConnectedActivePower == 1)
                                        {
                                            //发出功率以补充超出柴发最大发电能力的部分，或者在柴发最大发电能力范围内利用柴发多余电力给储能充电
                                            //1）、先确定微电网总有功功率设定值和总有功目标调节增量
                                            SystemOutput.Pset_Microgrid = -20;  //！！！后续需优化为配置量
                                            double Preal_Microgrid = 0;  //微电网实际功率
                                            for (size_t i = 0; i < IntegrationInput.size(); i = i + 1)
                                            {
                                                for (size_t j = 0; j < IntegrationInput[i].PCS.size(); j = j + 1)
                                                {
                                                    if (IntegrationInput[i].PCS[j].Type_PCS == 1)
                                                    {
                                                        Preal_Microgrid = Preal_Microgrid + IntegrationInput[i].PCS[j].P_real;
                                                    }
                                                }
                                            }
                                            SystemOutput.DeltaP_Microgrid = SystemOutput.Pset_Microgrid - Preal_Microgrid;  //输出：微网总有功目标调节增量

                                            //2）、进行增量分配，分为三种情况：
                                            if (SystemOutput.DeltaP_Microgrid > Configuration.DeadZone_IncreaseActivePower)	//微网总有功目标调节增量>升功率控制死区时，进行微网有功升功率控制
                                            {
                                                //微网升有功功率分配控制与有功设定值计算函数
                                                IncreaseActivePowerAllocateControlFunction(Configuration, IntegrationInput, SystemOutput, IntegrationOutput);
                                            }
                                            else if (SystemOutput.DeltaP_Microgrid < Configuration.DeadZone_DecreaseActivePower)  //微网总有功目标调节增量<降功率控制死区时，进行微网有功降功率控制
                                            {
                                                //微网降有功功率分配控制与有功设定值计算函数
                                                DecreaseActivePowerAllocateControlFunction(Configuration, IntegrationInput, SystemOutput, IntegrationOutput);
                                            }
                                            else if ((SystemOutput.DeltaP_Microgrid <= Configuration.DeadZone_IncreaseActivePower) && (SystemOutput.DeltaP_Microgrid >= Configuration.DeadZone_DecreaseActivePower))  //在升降功率控制死区范围内，微网有功不增不减，维持不变
                                            {
                                                SystemOutput.DeltaP_Microgrid = 0;
                                                for (size_t i = 0; i < IntegrationInput.size(); i = i + 1)
                                                {
                                                    if (IntegrationInput[i].PCS[j].Type_PCS == 1)
                                                    {
                                                        IntegrationOutput[i].PCS[j].ID_PCS = IntegrationInput[i].PCS[j].ID_PCS;
                                                        IntegrationOutput[i].PCS[j].deltaP_set = 0;
                                                        PCSActivePowerSetCalculateFunction(IntegrationInput[i].PCS[j], IntegrationOutput[i].PCS[j]);  //第i个风光储单元有功设定值的计算与修正，更新IntegrationOutput[i].PCS[j].P_set
                                                        SystemOutput.Pset_Microgrid = SystemOutput.Pset_Microgrid + IntegrationOutput[i].PCS[j].P_set;
                                                    }
                                                }
                                            }
                                        }
                                    }
                                    //情况2：只有市电作主电源微电网才并网运行，即要求市电状态与微网支路都正常，否则微网不对外发出或吸收功率
                                    else if (Configuration.Require_GridStatus == 1)
                                    {
                                        //微网不对外发出或吸收功率，控制所有未停机PCS的有功功率目标值为0
                                        for (size_t i = 0; i < IntegrationInput.size(); i = i + 1)
                                        {
                                            for (size_t j = 0; j < IntegrationInput[i].PCS.size(); j = j + 1)
                                            {
                                                if (IntegrationInput[i].PCS[j].Type_PCS == 1)
                                                {
                                                    if (IntegrationInput[i].PCS[j].RunStatus_PCS != 0)  //该PCS当前未处于停机状态
                                                    {
                                                        IntegrationOutput[i].PCS[j].ID_PCS = IntegrationInput[i].PCS[j].ID_PCS;
                                                        IntegrationOutput[i].PCS[j].deltaP_set = -IntegrationInput[i].PCS[j].P_real;
                                                        IntegrationOutput[i].PCS[j].P_set = 0;
                                                    }
                                                }
                                                else
                                                {
                                                    continue;
                                                }
                                            }
                                        }
                                    }
                                }
                            }
                        }
                        //作为主电源的PCS当前既不处于VF模式，也不处于PQ模式，因此需要将PCS切换至PQ模式运行
                        else
                        {
                            PCSPQModeControlFunction(IntegrationInput, IntegrationOutput);  //控制所有PCS处于PQ模式
                        }
                    }
                }
            }
        }
    }

    //无论是离网运行还是并网运行，都应该进行这一部分的控制。
    for (size_t i = 0; i < IntegrationInput.size(); i = i + 1)
    {
        //double RealPower_PCS = 0;  //双向变流器型PCS的实际功率
        double PowerSet_PCS = 0;  //双向变流器型PCS的总目标功率
        double SumRealPower_Wind_i = 0;  //风电总实时功率

        //1、统计双向变流器型PCS的总目标功率；并先控制风光启动、自由发电。
        for (size_t j = 0; j < IntegrationInput[i].PCS.size(); j = j + 1)
        {
            //统计双向变流器型PCS的总目标功率
            if (IntegrationInput[i].PCS[j].Type_PCS == 1)
            {
                //RealPower_PCS = RealPower_PCS + IntegrationInput[i].PCS[j].P_real;
                PowerSet_PCS = PowerSet_PCS + IntegrationOutput[i].PCS[j].P_set;
            }

            //先控制风电启动（即控制AC/DC处于VF模式，运行）、自由发电。
            if(IntegrationInput[i].LiEss.SOC_real < 90)
            {
                if (IntegrationInput[i].PCS[j].Type_PCS == 2)
                {
                    IntegrationOutput[i].PCS[j].ID_PCS = IntegrationInput[i].PCS[j].ID_PCS;
                    IntegrationOutput[i].PCS[j].RunModeControl_PCS = 1;  //输出：控制该PCS处于VF模式

                    if (IntegrationInput[i].PCS[j].RunMode_PCS == 1)
                    {
                        if (IntegrationInput[i].PCS[j].FaultStatus_PCS == 1)  //该PCS当前有故障，先尝试故障复位
                        {
                            IntegrationOutput[i].PCS[j].FaultResetControl_PCS = 1;  //输出：控制该PCS故障复位
                            continue; // 有故障未清除，不进行后续启停和模式切换控制
                        }
                        else if (IntegrationInput[i].PCS[j].FaultStatus_PCS == 0)  //该PCS当前无故障
                        {
                            IntegrationOutput[i].PCS[j].Num_FaultReset = 0;  //连续故障复位次数清零
                            IntegrationOutput[i].PCS[j].Flag_FaultResetFailure = 0;  //故障复位成功

                            IntegrationOutput[i].PCS[j].StartStopControl_PCS = 1;  //输出：控制该PCS启动
                        }
                    }
                }
            }
        }
        //控制光伏启动，自由发电
        IntegrationOutput[i].DCDC_PV.HighVoltage_set = 900;
        IntegrationOutput[i].DCDC_PV.Current_set = IntegrationInput[i].DCDC_PV.I_max;
        IntegrationOutput[i].DCDC_PV.P_set = IntegrationInput[i].DCDC_PV.Pn;
        if (IntegrationInput[i].DCDC_PV.FaultStatus_DCDC == 0)
        {
            IntegrationOutput[i].DCDC_PV.StartStopControl_DCDC = 1;

        }
        
        //2、根据SOC和PCS功率，更新风光控制
        for (size_t k = 0; k < IntegrationInput[i].Wind.size(); k = k + 1)
        {
            SumRealPower_Wind_i = SumRealPower_Wind_i + IntegrationInput[i].Wind[k].P_real;
        }

        //SOC>=SOCmax，或者储能禁充，PCS禁止充电，风光限发
        //if ((IntegrationInput[i].LiEss.SOC_real >= Configuration.SOCmax_LiEss) || (IntegrationInput[i].LiEss.Status == 1))
        if (IntegrationInput[i].LiEss.SOC_real >= Configuration.SOCmax_LiEss)
        {
            //PCS禁止充电，只能放电
            for (size_t j = 0; j < IntegrationInput[i].PCS.size(); j = j + 1)
            {
                if ((IntegrationInput[i].PCS[j].Type_PCS == 1) && (IntegrationOutput[i].PCS[j].P_set < 0))
                {
                    PowerSet_PCS = PowerSet_PCS - IntegrationOutput[i].PCS[j].P_set;
                    IntegrationOutput[i].PCS[j].P_set = 0;  //pcs禁充
                }
            }

            //控制风电停发
            for (size_t j = 0; j < IntegrationInput[i].PCS.size(); j = j + 1)
            {
                if (IntegrationInput[i].PCS[j].Type_PCS == 2)
                {
                    if ((IntegrationInput[i].PCS[j].RunStatus_PCS == 1) || (IntegrationOutput[i].PCS[j].StartStopControl_PCS == 1))
                    {
                        IntegrationOutput[i].PCS[j].ID_PCS = IntegrationInput[i].PCS[j].ID_PCS;
                        IntegrationOutput[i].PCS[j].StartStopControl_PCS = 0;  //全部风电停发
                    }
                }
            }

            //光伏限发
            IntegrationOutput[i].DCDC_PV.HighVoltage_set = IntegrationInput[i].LiEss.UnallowedChargingVoltage;  //此值在配置界面设为870
            IntegrationOutput[i].DCDC_PV.Current_set = IntegrationInput[i].DCDC_PV.I_max;
            IntegrationOutput[i].DCDC_PV.P_set = IntegrationInput[i].DCDC_PV.Pn;

            /*//风光功率>PCS功率，需限发（风电功率>PCS功率，风电停发，光伏功率=PCS功率；风电功率<=PCS功率，风电自由发电，光伏功率=PCS功率-风电功率）
            if ((SumRealPower_Wind_i + IntegrationInput[i].DCDC_PV.P_real) > PowerSet_PCS)  //！！！此处有问题，考虑是否应该用PCS的设定值？
            {
                if (SumRealPower_Wind_i > PowerSet_PCS)  //！！！此处有问题，考虑是否应该用PCS的设定值？
                {
                    for (size_t j = 0; j < IntegrationInput[i].PCS.size(); j = j + 1)
                    {
                        if (IntegrationInput[i].PCS[j].Type_PCS == 2)
                        {
                            if ((IntegrationInput[i].PCS[j].RunStatus_PCS == 1) || (IntegrationOutput[i].PCS[j].StartStopControl_PCS == 1))
                            {
                                IntegrationOutput[i].PCS[j].ID_PCS = IntegrationInput[i].PCS[j].ID_PCS;
                                IntegrationOutput[i].PCS[j].StartStopControl_PCS = 0;  //全部风电停发
                            }
                        }
                    }

                    IntegrationOutput[i].DCDC_PV.HighVoltage_set = IntegrationInput[i].DCDC_PV.HighVoltage_real;
                    IntegrationOutput[i].DCDC_PV.Current_set = IntegrationInput[i].DCDC_PV.I_max;
                    IntegrationOutput[i].DCDC_PV.P_set = PowerSet_PCS;  //光伏限发
                }
                else
                {
                    IntegrationOutput[i].DCDC_PV.HighVoltage_set = IntegrationInput[i].DCDC_PV.HighVoltage_real;
                    IntegrationOutput[i].DCDC_PV.Current_set = IntegrationInput[i].DCDC_PV.I_max;
                    IntegrationOutput[i].DCDC_PV.P_set = PowerSet_PCS - SumRealPower_Wind_i;  //风电自由发电，光伏限发
                }
            }
            else  //风光功率<=PCS功率，风电自由发电，光伏根据PCS功率限发
            {
                //光伏限发
                IntegrationOutput[i].DCDC_PV.HighVoltage_set = IntegrationInput[i].DCDC_PV.HighVoltage_real;
                IntegrationOutput[i].DCDC_PV.Current_set = IntegrationInput[i].DCDC_PV.I_max;
                IntegrationOutput[i].DCDC_PV.P_set = PowerSet_PCS - SumRealPower_Wind_i;
            }
            */
        }

        //储能SOC<=SOCmin，或者禁放，风光自由发电，限制PCS放电功率不超过风光发电功率
        //if ((IntegrationInput[i].LiEss.SOC_real <= Configuration.SOCmin_LiEss) || (IntegrationInput[i].LiEss.Status == 2))
        if (IntegrationInput[i].LiEss.SOC_real <= Configuration.SOCmin_LiEss)
        {
            for (size_t j = 0; j < IntegrationInput[i].PCS.size(); j = j + 1)
            {
                if (IntegrationInput[i].PCS[j].Type_PCS == 1)
                {
                    if (IntegrationOutput[i].PCS[j].P_set > (SumRealPower_Wind_i + IntegrationInput[i].DCDC_PV.P_real))
                    {
                        IntegrationOutput[i].PCS[j].ID_PCS = IntegrationInput[i].PCS[j].ID_PCS;
                        IntegrationOutput[i].PCS[j].P_set = SumRealPower_Wind_i + IntegrationInput[i].DCDC_PV.P_real;  //PCS放电功率不能超过风光功率
                    }
                }
            }

            //储能电池自充电保护
            bool Flag_Autocharge = false;
            if (IntegrationInput[i].LiEss.SOC_real < Configuration.SOC_Autocharge)
            {
                Flag_Autocharge = true;
            }
            if (IntegrationInput[i].LiEss.SOC_real >= Configuration.SOC_StopAutocharge)  //有问题，无法起效
            {
                Flag_Autocharge = false;
            }

            if (Flag_Autocharge)
            {
                for (size_t j = 0; j < IntegrationInput[i].PCS.size(); j = j + 1)
                {
                    if (IntegrationInput[i].PCS[j].Type_PCS == 1)
                    {
                        if (IntegrationOutput[i].PCS[j].P_set > 0)
                        {
                            IntegrationOutput[i].PCS[j].ID_PCS = IntegrationInput[i].PCS[j].ID_PCS;
                            IntegrationOutput[i].PCS[j].P_set = -10;  //PCS充电
                        }
                    }
                }
            }
        }

        //储能待机或停机，储能无法充放，DCDC输出功率为0，AC/DC停机，PCS功率为0
        if ((IntegrationInput[i].LiEss.Status == 3) || (IntegrationInput[i].LiEss.Status == 4))
        {
            for (size_t j = 0; j < IntegrationInput[i].PCS.size(); j = j + 1)
            {
                if ((IntegrationInput[i].PCS[j].RunStatus_PCS == 1) || (IntegrationOutput[i].PCS[j].StartStopControl_PCS == 1))
                {
                    IntegrationOutput[i].PCS[j].ID_PCS = IntegrationInput[i].PCS[j].ID_PCS;
                    IntegrationOutput[i].PCS[j].StartStopControl_PCS = 0;
                    IntegrationOutput[i].PCS[j].P_set = 0;
                }               
            }

            IntegrationOutput[i].DCDC_PV.HighVoltage_set = 0;
            IntegrationOutput[i].DCDC_PV.Current_set = 0;
            IntegrationOutput[i].DCDC_PV.P_set = 0;  //风光停发，PCS停机
        }

        //3、根据液冷机组状态，更新DC/DC控制
        if (IntegrationInput[i].LiEss.TMSComState == 170)
        {
            if (IntegrationInput[i].LiEss.ClusterMaxCellTemp < 75)  //有40的温度偏移，实际为35摄氏度
            {
                IntegrationOutput[i].DCDC_PV.Current_set = 20;
            }
            else if (IntegrationInput[i].LiEss.ClusterMaxCellTemp < 80)
            {
                IntegrationOutput[i].DCDC_PV.Current_set = 10;
            }
            else if (IntegrationInput[i].LiEss.ClusterMaxCellTemp < 85)
            {
                IntegrationOutput[i].DCDC_PV.Current_set = 5;
            }
            else if(IntegrationInput[i].LiEss.ClusterMaxCellTemp >= 85)
            {
                IntegrationOutput[i].DCDC_PV.Current_set = 0;
            }
        }

        //统计微电网总有功功率设定值Pset_Microgrid
        for (size_t j = 0; j < IntegrationInput[i].PCS.size(); j = j + 1)
        {
            if (IntegrationInput[i].PCS[j].Type_PCS == 1)
            {
                SystemOutput.Pset_Microgrid = SystemOutput.Pset_Microgrid + IntegrationOutput[i].PCS[j].P_set;
            }
        }
    }
    /*
    for (size_t i = 0; i < IntegrationInput.size(); i = i + 1)
    {
        if (IntegrationInput[i].LiEss.TMSComState == 170)
        {
            if (IntegrationInput[i].LiEss.ClusterMaxCellTemp < 75)  //有40的温度偏移，实际为35摄氏度
            {
                IntegrationOutput[i].DCDC_PV.Current_set = 20;
            }
            else if (IntegrationInput[i].LiEss.ClusterMaxCellTemp < 80)
            {
                IntegrationOutput[i].DCDC_PV.Current_set = 10;
            }
            else if (IntegrationInput[i].LiEss.ClusterMaxCellTemp < 85)
            {
                IntegrationOutput[i].DCDC_PV.Current_set = 5;
            }
            else if(IntegrationInput[i].LiEss.ClusterMaxCellTemp >= 85)
            {
                IntegrationOutput[i].DCDC_PV.Current_set = 0;
            }
        }
    }
    */
    return true;
}

//外部电网状态判断函数
void HaikouMicrogridEMSAlgorithm::ExternalPowerGridStatusJudgmentFunction(Configuration_Input Configuration, ExternalPowerGrid_Input ExternalPowerGrid, vector<Switch_Input> Switch, System_Output& SystemOutput)
{
    SystemOutput.Status_GridPCC = 1;  //对市电状态进行初始化
    SystemOutput.Status_MicrogridBranch = 1;  //对微网支路状态进行初始化

    //对市电状态进行判断
    if ((fabs(ExternalPowerGrid.Ua_Grid - Configuration.Un_Grid) <= Configuration.AllowableDeviation_Ugrid) && (fabs(ExternalPowerGrid.Ub_Grid - Configuration.Un_Grid) <= Configuration.AllowableDeviation_Ugrid) && (fabs(ExternalPowerGrid.Uc_Grid - Configuration.Un_Grid) <= Configuration.AllowableDeviation_Ugrid))
    {
        //市电开关状态判断
        for (size_t i = 0;i < Switch.size();i = i + 1)
        {
            if (Switch[i].Type == 0)  //如果是市电并网点开关
            {
                if (Switch[i].FaultFlag == 0)  //该开关当前无故障
                {
                    if (Switch[i].Status != 1)  //该开关当前未处于合闸状态
                    {
                        SystemOutput.Status_GridPCC = 0;  //输出：市电不正常
                        break;
                    }
                    else
                    {
                        continue;  //该市电并网点开关闭合，跳出当前迭代，开始下一次迭代
                    }
                }
                else if (Switch[i].FaultFlag == 1)  //该开关当前有故障
                {
                    SystemOutput.Status_GridPCC = 0;  //输出：市电不正常
                    break;
                }
            }
            else  //不是市电并网点开关，跳出当前迭代，开始下一次迭代
            {
                continue;
            }
        }
    }
    else
    {
        SystemOutput.Status_GridPCC = 0;  //输出：市电不正常
    }

    //对微网支路状态进行判断
    //微网接入点三相电压幅值偏差判断
    if ((fabs(ExternalPowerGrid.Ua_Microgrid - Configuration.Un_Microgrid) <= Configuration.AllowableDeviation_Umicrogrid) && (fabs(ExternalPowerGrid.Ub_Microgrid - Configuration.Un_Microgrid) <= Configuration.AllowableDeviation_Umicrogrid) && (fabs(ExternalPowerGrid.Uc_Microgrid - Configuration.Un_Microgrid) <= Configuration.AllowableDeviation_Umicrogrid))
    {
        //微网支路开关状态判断
        for (size_t i = 0;i < Switch.size();i = i + 1)
        {
            if (Switch[i].Type == 1)  //如果是微网支路开关
            {
                if (Switch[i].FaultFlag == 0)  //该开关当前无故障
                {
                    if (Switch[i].Status != 1)  //该开关当前未处于合闸状态
                    {
                        SystemOutput.Status_MicrogridBranch = 0;  //输出：微网支路不正常
                        break;
                    }
                    else
                    {
                        continue;  //该微网支路开关闭合，跳出当前迭代，开始下一次迭代
                    }
                }
                else if (Switch[i].FaultFlag == 1)  //该开关当前有故障
                {
                    SystemOutput.Status_MicrogridBranch = 0;  //输出：微网支路不正常
                    break;
                }
            }
            else  //不是微网支路开关，跳出当前迭代，开始下一次迭代
            {
                continue;
            }
        }
    }
    else  //微网接入点三相电压幅值偏差没有都在允许范围内
    {
        SystemOutput.Status_MicrogridBranch = 0;  //输出：微网支路不正常
    }
}

//将微电网中所有需工作于VF模式、无设备故障的、用于连接负荷与风光储的双向变流器型PCS切换至VF模式运行
void HaikouMicrogridEMSAlgorithm::PCSVFModeControlFunction(vector<Integration_Input> IntegrationInput, vector<Integration_Output>& IntegrationOutput)
{
    for (size_t i = 0; i < IntegrationInput.size(); i = i + 1)
    {
        for (size_t j = 0; j < IntegrationInput[i].PCS.size(); j = j + 1)
        {
            IntegrationOutput[i].PCS[j].ID_PCS = IntegrationInput[i].PCS[j].ID_PCS;

            if (IntegrationInput[i].PCS[j].Type_PCS == 1)  //该PCS为用于连接负荷与风光储的双向变流器型PCS
            {
                if (IntegrationInput[i].PCS[j].Flag_VFPCS == 1)  //需将该PCS控制于VF模式
                {
                    if (IntegrationInput[i].PCS[j].RunMode_PCS != 1)  //该PCS当前未工作于VF模式
                    {
                        IntegrationOutput[i].PCS[j].RunModeControl_PCS = 1;  //输出：控制该PCS处于VF模式
                    }
                    else if (IntegrationInput[i].PCS[j].RunMode_PCS == 1)
                    {
                        if (IntegrationInput[i].PCS[j].FaultStatus_PCS == 1)  //该PCS当前有故障，先尝试故障复位
                        {
                            IntegrationOutput[i].PCS[j].Num_FaultReset = IntegrationOutput[i].PCS[j].Num_FaultReset + 1;  //连续故障复位次数累计
                            if (IntegrationOutput[i].PCS[j].Num_FaultReset <= 2)
                            {
                                IntegrationOutput[i].PCS[j].FaultResetControl_PCS = 1;  //输出：控制该PCS故障复位
                                // 注意：实际系统中，复位指令可能需要在下个周期清除，这里只是发送复位命令，复位是否成功需要看下个周期的FaultStatus_PCS

                                // 复位失败状态判断
                                if (IntegrationOutput[i].PCS[j].Num_FaultReset == 1)
                                {
                                    IntegrationOutput[i].PCS[j].Flag_FaultResetFailure = 0; // 期望第一次故障复位成功
                                }
                                else if (IntegrationOutput[i].PCS[j].Num_FaultReset == 2)
                                {
                                    IntegrationOutput[i].PCS[j].Flag_FaultResetFailure = 1; // 第一次故障复位失败
                                }
                                continue; // 有故障未清除，不进行后续启停和模式切换控制
                            }
                            else
                            {
                                // 故障复位次数超过2次，不再尝试复位
                                IntegrationOutput[i].PCS[j].FaultResetControl_PCS = 0;
                                IntegrationOutput[i].PCS[j].Flag_FaultResetFailure = 2; // 复位失败状态判断：连续故障复位失败
                                continue; // 有故障未清除，不进行后续启停和模式切换控制
                            }
                        }
                        else if (IntegrationInput[i].PCS[j].FaultStatus_PCS == 0)  //该PCS当前无故障
                        {
                            IntegrationOutput[i].PCS[j].Num_FaultReset = 0;  //连续故障复位次数清零
                            IntegrationOutput[i].PCS[j].Flag_FaultResetFailure = 0;  //故障复位成功

                            if (IntegrationInput[i].PCS[j].RunStatus_PCS == 0)  //该PCS当前处于停机状态
                            {
                                IntegrationOutput[i].PCS[j].StartStopControl_PCS = 1;  //输出：控制该PCS启动
                                continue;
                            }
                        }
                    }
                }
                else
                {
                    continue;
                }
            }
            else
            {
                continue;
            }
        }
    }
}

//将微电网中所有无设备故障的PCS切换至PQ模式运行
void HaikouMicrogridEMSAlgorithm::PCSPQModeControlFunction(vector<Integration_Input> IntegrationInput, vector<Integration_Output>& IntegrationOutput)
{
    for (size_t i = 0; i < IntegrationInput.size(); i = i + 1)
    {
        for (size_t j = 0; j < IntegrationInput[i].PCS.size(); j = j + 1)
        {
            IntegrationOutput[i].PCS[j].ID_PCS = IntegrationInput[i].PCS[j].ID_PCS;

            if (IntegrationInput[i].PCS[j].Type_PCS == 1)  //该PCS为用于连接负荷与风光储的双向变流器型PCS
            {
                if (IntegrationInput[i].PCS[j].FaultStatus_PCS == 1)  //该PCS当前有故障
                {
                    continue;  //有故障未清除，跳出当前循环，不进行后续启停和模式切换控制
                }
                else  //该PCS当前无故障
                {
                    IntegrationOutput[i].PCS[j].Num_FaultReset = 0;  //连续故障复位次数清零
                    IntegrationOutput[i].PCS[j].Flag_FaultResetFailure = 0;  //故障复位成功

                    if (IntegrationInput[i].PCS[j].RunMode_PCS != 2)  //该PCS当前未工作于PQ模式
                    {
                        IntegrationOutput[i].PCS[j].RunModeControl_PCS = 2;  //输出：控制该PCS处于PQ模式
                    }
                    else if(IntegrationInput[i].PCS[j].RunMode_PCS == 2)
                    {
                        IntegrationOutput[i].PCS[j].StartStopControl_PCS = 1;  //输出：控制该PCS启动

                    }
                }
            }
            else
            {
                continue;
            }
        }
    }
}

//微网供电能力判断函数
void HaikouMicrogridEMSAlgorithm::MicrogridPowerSupplyCapacityJudgmentFunction(Configuration_Input Configuration,vector<Integration_Input> IntegrationInput, System_Output& SystemOutput)
{
    SystemOutput.Flag_MicrogridSupply1 = 0;  // 初始化微网供电能力标志位为0（能力不足）
	SystemOutput.Flag_MicrogridSupply2 = 0;

	double Pmax_Integration_i = 0;  //某一体化单元的当前最大可输出功率
	double DischargeEnergy_Integration_i = 0;  //某一体化单元的当前最大可放出总电量
	double Pmax_Microgrid = 0;  //微电网的当前最大可输出总功率
	double DischargeEnergy_Microgrid = 0;  //微电网的当前最大可放出总电量

	bool foundVFPCS = false;  // 微电网中是否存在能作为主电源的PCS
	bool isInOffgridMode = false;  // 微网是否正在离网运行

	//第一步：计算微电网当前所能输出的最大总功率，以及所能放出的最大总电量（只考虑储能电池中储备的电量）
	for (size_t i = 0;i < IntegrationInput.size();i = i + 1)
	{
        for (size_t j = 0; j < IntegrationInput[i].PCS.size(); j = j + 1)
        {
            if (IntegrationInput[i].PCS[j].Type_PCS == 1)  //该PCS为用于连接负荷与风光储的双向变流器型PCS
            {
                if (IntegrationInput[i].LiEss.SOC_real > Configuration.SOCmin_LiEss)  //该PCS当前SOC大于SOCmin
                {
                    Pmax_Integration_i = IntegrationInput[i].PCS[j].P_max;
                    DischargeEnergy_Integration_i = 0.01 * (IntegrationInput[i].LiEss.SOC_real - Configuration.SOCmin_LiEss) * IntegrationInput[i].LiEss.RatedCapacity;
                }
                else
                {
                    Pmax_Integration_i = 0;
                    DischargeEnergy_Integration_i = 0;
                }
            }
            else
            {
                continue;
            }
            Pmax_Microgrid = Pmax_Microgrid + Pmax_Integration_i;
            DischargeEnergy_Microgrid = DischargeEnergy_Microgrid + DischargeEnergy_Integration_i;

            // 同时检查微电网是否存在能作为主电源的PCS及其当前运行模式
            if (IntegrationInput[i].PCS[j].Type_PCS == 1 && IntegrationInput[i].PCS[j].Flag_VFPCS == 1)
            {
                foundVFPCS = true;

                if (IntegrationInput[i].PCS[j].RunMode_PCS == 1 && IntegrationInput[i].PCS[j].RunStatus_PCS == 1)
                {
                    isInOffgridMode = true;  // 该PCS处于VF模式，表示微网正在离网运行
                }
            }
        }
	}
	
	// 第二步：判断微网供电能力
	if (Pmax_Microgrid >= Configuration.P_max_Load)
	{
		if (foundVFPCS == true)
		{
			if (isInOffgridMode == true)  //作为主电源的PCS已经处于VF模式，即代表微网正在离网运行
			{
				if (DischargeEnergy_Microgrid > 0)
				{
					SystemOutput.Flag_MicrogridSupply2 = 1;
                    SystemOutput.Flag_MicrogridSupply1 = 1;
				}
				else
				{
					SystemOutput.Flag_MicrogridSupply2 = 0;
                    SystemOutput.Flag_MicrogridSupply1 = 0;
				}
			}
			else if (isInOffgridMode == false)  //作为主电源的PCS未处于VF模式，即代表微网没有在离网运行
			{
				if (DischargeEnergy_Microgrid >= Configuration.P_average_Load * Configuration.Time_OffgridOperation)  //!!!注意：此处可能需优化，平均功率按白天与晚上进行区分计算
				{
					SystemOutput.Flag_MicrogridSupply1 = 1;
                    SystemOutput.Flag_MicrogridSupply2 = 1;
				}
				else
				{
					SystemOutput.Flag_MicrogridSupply1 = 0;
                    SystemOutput.Flag_MicrogridSupply2 = 0;
				}
			}
		}
	}
	else
	{
		SystemOutput.Flag_MicrogridSupply1 = 0;
		SystemOutput.Flag_MicrogridSupply2 = 0;
	}
}

//微网并网有功功率控制函数
void HaikouMicrogridEMSAlgorithm::GridConnectedActivePowerControlFunction(Configuration_Input Configuration, ExternalPowerGrid_Input ExternalPowerGrid, vector<Integration_Input> IntegrationInput, System_Output& SystemOutput, vector<Integration_Output>& IntegrationOutput)
{
    DeltaPCalculationFunction(Configuration, ExternalPowerGrid, IntegrationInput, SystemOutput);  //微网总有功目标调节增量计算，得到当前周期的待控制偏差量

    //增加有功闭环修正控制
    double ControlDeviation_1 = ExternalPowerGrid.P_real_pcc - ExternalPowerGrid.P_previousone_pcc - ExternalPowerGrid.DeltaP_Microgrid_1;  //上一周期控制误差
    SystemOutput.DeltaP_Microgrid = Configuration.Kp_ActivePowerControl * (SystemOutput.DeltaP_Microgrid - ControlDeviation_1) + Configuration.Ki_ActivePowerControl * SystemOutput.DeltaP_Microgrid * Configuration.ControlCycle;
    //double SumControlDeviation = ExternalPowerGrid.P_previousthree_pcc - ExternalPowerGrid.P_real_pcc - ExternalPowerGrid.DeltaP_Microgrid_1 - ExternalPowerGrid.DeltaP_Microgrid_2 - ExternalPowerGrid.DeltaP_Microgrid_3;
    //SystemOutput.DeltaP_Microgrid = SystemOutput.DeltaP_Microgrid + Configuration.Kp_ActivePowerControl * ControlDeviation_1 + Configuration.Ki_ActivePowerControl * SumControlDeviation;

    if (SystemOutput.DeltaP_Microgrid > Configuration.DeadZone_IncreaseActivePower)	//微网总有功目标调节增量>升功率控制死区时，进行微网有功升功率控制
    {
        //微网升有功功率分配控制与有功设定值计算函数
        IncreaseActivePowerAllocateControlFunction(Configuration, IntegrationInput, SystemOutput, IntegrationOutput);
    }

    else if (SystemOutput.DeltaP_Microgrid < Configuration.DeadZone_DecreaseActivePower)  //微网总有功目标调节增量<降功率控制死区时，进行微网有功降功率控制
    {
        //微网降有功功率分配控制与有功设定值计算函数
        DecreaseActivePowerAllocateControlFunction(Configuration, IntegrationInput, SystemOutput, IntegrationOutput);
    }
    else if ((SystemOutput.DeltaP_Microgrid <= Configuration.DeadZone_IncreaseActivePower) && (SystemOutput.DeltaP_Microgrid >= Configuration.DeadZone_DecreaseActivePower))  //在升降功率控制死区范围内，微网有功不增不减，维持不变
    {
        SystemOutput.DeltaP_Microgrid = 0;
        SystemOutput.Flag_ExceedMicrogridCapability = 0;
        for (size_t i = 0;i < IntegrationInput.size();i = i + 1)
        {
            for (size_t j = 0; j < IntegrationInput[i].PCS.size(); j = j + 1)
            {
                if (IntegrationInput[i].PCS[j].Type_PCS == 1)  //该PCS为用于连接负荷与风光储的双向变流器型PCS
                {
                    IntegrationOutput[i].PCS[j].ID_PCS = IntegrationInput[i].PCS[j].ID_PCS;
                    IntegrationOutput[i].PCS[j].deltaP_set = 0;
                    PCSActivePowerSetCalculateFunction(IntegrationInput[i].PCS[j], IntegrationOutput[i].PCS[j]);  //第i个风光储单元有功设定值的计算与修正，更新IntegrationOutput[i].PCS[j].P_set
                }
                else
                {
                    continue;
                }
            }
        }
    }
}

//微电网并网运行总有功目标调节增量计算函数
void HaikouMicrogridEMSAlgorithm::DeltaPCalculationFunction(Configuration_Input Configuration, ExternalPowerGrid_Input ExternalPowerGrid, vector<Integration_Input> IntegrationInput, System_Output& SystemOutput)
{
    //SystemOutput.Flag_OverWarning = 0;  // 初始化超限预警标志位
    double P_warn_G2M = Configuration.P_restrict_G2M + Configuration.DeltaP_warn_G2M;  //电网向微电网下送功率即将超限的预警值=限制值+偏差量
    if (P_warn_G2M > 0)
    {
        P_warn_G2M = 0;
    }
    double P_warn_M2G = Configuration.P_restrict_M2G - Configuration.DeltaP_warn_M2G;  //微电网向电网上送功率即将超限的预警值=限制值-偏差量
    //一、更新 SystemOutput.P0_max、 SystemOutput.P0_min、SystemOutput.num1_conwarnning、SystemOutput.num2_conwarnning
    //1、当前周期的交换功率在下送预警值与上送预警值之间
    if ((ExternalPowerGrid.P_real_pcc >= P_warn_G2M) && (ExternalPowerGrid.P_real_pcc <= P_warn_M2G))
    {
        SystemOutput.Flag_OverWarning = 0;  // 当前周期未超预警，超限预警标志位置0
        SystemOutput.P0_max = P_warn_M2G;  //上限控制参考值=上送预警值
        SystemOutput.num1_conwarnning = 0;  //上限控制连续预警周期=0
        SystemOutput.P0_min = P_warn_G2M;  //下限控制参考值=下送预警值
        SystemOutput.num2_conwarnning = 0;  //下限控制连续预警周期=0
    }
    //2、当前周期的交换功率大于上送预警值
    else if (ExternalPowerGrid.P_real_pcc > P_warn_M2G)
    {
        SystemOutput.P0_min = P_warn_G2M;  //下限控制参考值=下送预警值
        SystemOutput.num2_conwarnning = 0;  //下限控制连续预警周期=0

        SystemOutput.Flag_OverWarning = 1;  //当前周期超上限预警，超限预警标志位置1

        //2.1 当前周期超上限预警，但上一周期没有超上限预警，即当前周期是第一次超上限预警
        if (ExternalPowerGrid.Flag_OverWarning_1 != 1)
        {
            SystemOutput.P0_max = ExternalPowerGrid.P_real_pcc;  //第一次超上限预警，上限控制参考值=当前实时交换功率
            SystemOutput.num1_conwarnning = 1;  //上限控制连续预警周期=1
        }
        //2.2 上一周期与当前周期均超上限预警
        else if (ExternalPowerGrid.Flag_OverWarning_1 == 1)
        {
            SystemOutput.num1_conwarnning = SystemOutput.num1_conwarnning + 1;  //上限控制连续预警周期加1

            if (ExternalPowerGrid.P_real_pcc > ExternalPowerGrid.P0_max_1)  //如果当前实时交换功率>前一周期的P0_max
            {
                SystemOutput.P0_max = ExternalPowerGrid.P_real_pcc;  //更新上限控制参考值=当前实时交换功率
            }
            else
            {
                SystemOutput.P0_max = ExternalPowerGrid.P0_max_1;  //更新上限控制参考值=前一周期的P0_max
            }
        }
    }
    //3、当前周期的交换功率小于下送预警值
    else if (ExternalPowerGrid.P_real_pcc < P_warn_G2M)  //当前周期的交换功率小于下送预警值
    {
        SystemOutput.P0_max = P_warn_M2G;  //上限控制参考值=上送预警值
        SystemOutput.num1_conwarnning = 0;  //上限控制连续预警周期=0

        SystemOutput.Flag_OverWarning = 2;  //超下限预警，超限预警标志位置2

        //3.1 当前周期超下限预警，但上一周期没有超下限预警，即当前周期是第一次超下限预警
        if (ExternalPowerGrid.Flag_OverWarning_1 != 2)
        {
            SystemOutput.P0_min = ExternalPowerGrid.P_real_pcc;  //第一次超下限预警，下限控制参考值=当前实时交换功率
            SystemOutput.num2_conwarnning = 1;  //下限控制连续预警周期=1
        }
        //3.2 上一周期与当前周期均超下限预警
        else if (ExternalPowerGrid.Flag_OverWarning_1 == 2)
        {
            SystemOutput.num2_conwarnning = SystemOutput.num2_conwarnning + 1;  //下限控制连续预警周期加1

            if (ExternalPowerGrid.P_real_pcc < ExternalPowerGrid.P0_min_1)  //如果当前实时交换功率<下限控制参考值
            {
                SystemOutput.P0_min = ExternalPowerGrid.P_real_pcc;  //更新下限控制参考值=当前实时交换功率
            }
            else
            {
                SystemOutput.P0_min = ExternalPowerGrid.P0_min_1;  //更新上限控制参考值=前一周期的P0_max
            }
        }
    }

    //二、根据交换功率大小分工况进行控制：超上限控制、超下限控制、经济调度控制。
    //1）实时交换功率介于下送功率超限预警值与上送功率超限预警值之间，进行经济调度控制。
    if ((ExternalPowerGrid.P_real_pcc > P_warn_G2M) && (ExternalPowerGrid.P_real_pcc < P_warn_M2G))
    {
        //判断是否启用峰谷套利控制功能
        if (Configuration.ControlFlag_EconomicDispatch == 1)
        {
            EconomicDispatchControlFunction1(Configuration, ExternalPowerGrid, IntegrationInput, SystemOutput);  //微电网峰谷套利模式下的并网经济调度控制函数
        }
        else if (Configuration.ControlFlag_EconomicDispatch == 0)
        {
            SystemOutput.DeltaP_Microgrid = P_warn_M2G - ExternalPowerGrid.P_real_pcc;  //增加微网输出功率，此时DeltaP_Microgrid>0
        }
    }
    //2）实时交换功率大于上送功率超限预警值，进行超上限控制。
    else if (ExternalPowerGrid.P_real_pcc > P_warn_M2G)
    {
        //实时交换功率大于等于上送功率限制值，立即进行超限控制
        if (ExternalPowerGrid.P_real_pcc >= Configuration.P_restrict_M2G)
        {
            SystemOutput.DeltaP_Microgrid = P_warn_M2G - ExternalPowerGrid.P_real_pcc;  //减小微网输出功率，此时DeltaP_Microgrid<0
        }
        //实时交换功率介于上送功率预警值与上送功率限制值之间
        else
        {
            //上限控制持续预警周期超过允许的最大周期（长期超过预警值）或者交换功率呈现逼近上送功率限制值的趋势，立即进行超限控制
            if ((SystemOutput.num1_conwarnning > Configuration.Maxnum_conwarnning) || ((ExternalPowerGrid.Flag_OverWarning_1 == 1) && (ExternalPowerGrid.P_real_pcc >= SystemOutput.P0_max)))
            {
                SystemOutput.DeltaP_Microgrid = P_warn_M2G - ExternalPowerGrid.P_real_pcc;  //减小微网输出功率，此时DeltaP_Microgrid<0
            }
            else
            {
                SystemOutput.DeltaP_Microgrid = 0;
            }
        }
    }
    //3）实时交换功率小于下送功率预警值，进行超下限控制。
    else if (ExternalPowerGrid.P_real_pcc < P_warn_G2M)
    {
        //实时交换功率小于等于下送功率限制值，立即进行超限控制
        if (ExternalPowerGrid.P_real_pcc <= Configuration.P_restrict_G2M)
        {
            SystemOutput.DeltaP_Microgrid = P_warn_G2M - ExternalPowerGrid.P_real_pcc;  //增加微网输出功率，此时DeltaP_Microgrid>0
        }
        //实时交换功率介于下送功率预警值与下送功率限制值之间
        else
        {
            //下限控制持续预警周期超过允许的最大周期（长期超过预警值）或者交换功率呈现逼近下送功率限制值的趋势，立即进行超限控制
            if ((SystemOutput.num2_conwarnning > Configuration.Maxnum_conwarnning) || ((ExternalPowerGrid.Flag_OverWarning_1 == 2) && (ExternalPowerGrid.P_real_pcc <= SystemOutput.P0_min)))
            {
                SystemOutput.DeltaP_Microgrid = P_warn_G2M - ExternalPowerGrid.P_real_pcc;  //增加微网输出功率，此时DeltaP_Microgrid>0
            }
            else
            {
                SystemOutput.DeltaP_Microgrid = 0;
            }
        }
    }
}

//微电网峰谷套利模式下的并网经济调度控制函数
void HaikouMicrogridEMSAlgorithm::EconomicDispatchControlFunction1(Configuration_Input Configuration, ExternalPowerGrid_Input ExternalPowerGrid, vector<Integration_Input> IntegrationInput, System_Output& SystemOutput)
{
    double SumRemainingCapacity_RunLiEss = 0;  //微电网内全部还能放电的储能电池当前所存储的电量
    double SumRatedCapacity_RunLiEss = 0;  //微电网内全部运行中的储能电池的总额定电量
    double SumMinPower_RunLiEss = 0;  //微电网内全部还能充电的储能电池的总最小运行功率，即总最大可充电功率。
    double SumRealPower_Wind = 0;  //微电网内全部风电的当前总实时有功功率
    double SumRealPower_PV = 0;  //微电网内全部光伏的当前总实时有功功率
    double SumRealPower_PCS = 0;  //微电网内全部PCS的当前总实时有功功率

    for (size_t i = 0; i < IntegrationInput.size(); i = i + 1)
    {
        //统计微电网内全部还能放电的储能电池当前所存储的电量
        if ((IntegrationInput[i].LiEss.Status == 0) || (IntegrationInput[i].LiEss.Status == 1))
        {
            SumRemainingCapacity_RunLiEss = SumRemainingCapacity_RunLiEss + 0.01 * IntegrationInput[i].LiEss.SOC_real * IntegrationInput[i].LiEss.RatedCapacity;
        }
        //统计微电网内全部运行中的储能电池的总额定电量
        if ((IntegrationInput[i].LiEss.Status == 0) || (IntegrationInput[i].LiEss.Status == 1) || (IntegrationInput[i].LiEss.Status == 2))
        {
            SumRatedCapacity_RunLiEss = SumRatedCapacity_RunLiEss + IntegrationInput[i].LiEss.RatedCapacity;
        }
        //统计微电网内全部还能充电的储能电池的总最小运行功率，即总最大可充电功率。
        if ((IntegrationInput[i].LiEss.Status == 0) || (IntegrationInput[i].LiEss.Status == 2))
        {
            SumMinPower_RunLiEss = SumMinPower_RunLiEss + IntegrationInput[i].LiEss.P_min;
        }
        //统计微电网内全部风电的当前总实时有功功率
        for (size_t j = 0; j < IntegrationInput[i].Wind.size(); j = j + 1)
        {
            SumRealPower_Wind = SumRealPower_Wind + IntegrationInput[i].Wind[j].P_real;
        }
        //统计微电网内全部光伏的当前总实时有功功率
        SumRealPower_PV = SumRealPower_PV + IntegrationInput[i].DCDC_PV.P_real;

        //统计微电网内全部PCS的当前总实时有功功率
        for (size_t k = 0; k < IntegrationInput[i].PCS.size(); k = k + 1)
        {
            if (IntegrationInput[i].PCS[k].Type_PCS == 1)  //该PCS为用于连接负荷与风光储的双向变流器型PCS
            {
                SumRealPower_PCS = SumRealPower_PCS + IntegrationInput[i].PCS[k].P_real;
            }
            else
            {
                continue;
            }
        }
    }

    double SumRequiredCapacity_RunLiEss = 0;  //峰谷套利模式下，为了第二天尽可能少使用平段与峰段高电价时段的下网电量，全部运行中的储能电池所需储备的电量
    //微电网所在地区第二天的天气情况无法获取，储能所需储备电量=峰谷套利经济调度控制模式下的储能电池SOC默认值*额定容量
    if (Configuration.Flag_WeatherStatus == 0)
    {
        for (size_t i = 0; i < IntegrationInput.size(); i = i + 1)
        {
            if ((IntegrationInput[i].LiEss.Status == 0) || (IntegrationInput[i].LiEss.Status == 1) || (IntegrationInput[i].LiEss.Status == 2))
            {
                SumRequiredCapacity_RunLiEss = SumRequiredCapacity_RunLiEss + 0.01 * Configuration.InitialSOC_EconomicDispatch * IntegrationInput[i].LiEss.RatedCapacity;
            }
        }
    }
    else  //如果微电网所在地区第二天的天气情况可以获取，不同天气情况下全部运行中的储能电池所需储备的电量不同
    {
        // 根据第二天天气情况，确定储能储备电量所需供应负荷用电时长
        double RequiredHours = 0;
        switch (ExternalPowerGrid.WeatherStatus)
        {
        case 0: RequiredHours = Configuration.Time_Rainy; break;   //阴雨天，储能在保障最低荷电状态的基础上，还需储备能供应负荷Time_Rainy（海口微网项目默认12h）时长用电的电量
        case 1: RequiredHours = Configuration.Time_Cloudy; break;  //多云天，储能在保障最低荷电状态的基础上，还需储备能供应负荷Time_Cloudy（海口微网项目默认7h）时长用电的电量
        case 2: RequiredHours = Configuration.Time_Sunny; break;   //晴天，储能在保障最低荷电状态的基础上，还需储备能供应负荷Time_Sunny （海口微网项目默认1.5h）时长用电的电量
        default: RequiredHours = 0; break;
        }

        // 所需储备电量 = 负荷需求 + 最小SOC保障
        SumRequiredCapacity_RunLiEss = Configuration.P_average_Load * RequiredHours + Configuration.SOCmin_LiEss * SumRatedCapacity_RunLiEss;
    }

    if (SumRequiredCapacity_RunLiEss > SumRatedCapacity_RunLiEss)
    {
        SumRequiredCapacity_RunLiEss = SumRatedCapacity_RunLiEss;
    }

    //根据当前处于不同的峰平谷分时时段，分别进行控制
    double SumDeltaCapacity_RunLiEss = 0;  //微电网内全部运行中的储能电池所需放出或充入的电量总变化增量（单位：kWh。放出电量时，总变化增量为正值；充入电量时，总变化增量为负值）
    double SumPowerSet_RunLiEss = 0;  //微电网内全部运行中的储能电池的总功率（单位：kW。放电时，功率为正值；充电时，功率为负值）
    const double Time_Low = 8;  //低谷电价时段的时长（单位：h）
    if (ExternalPowerGrid.TimeStatus != 0)  //当前不处于低谷电价时段，增加微网输出功率
    {
        SystemOutput.DeltaP_Microgrid = Configuration.P_restrict_M2G - Configuration.DeltaP_warn_M2G - ExternalPowerGrid.P_real_pcc;  //增加微网输出功率，此时DeltaP_Microgrid>0
    }
    else  //当前处于低谷电价时段，对比储能当前实际电量SumRemainingCapacity_RunLiEss与所需储备的电量SumRequiredCapacity_RunLiEss之间的大小关系，分情况进行控制
    {
        if (SumRemainingCapacity_RunLiEss < SumRequiredCapacity_RunLiEss)  //储能电池当前电量不足，需充电增加储备电量
        {
            SumDeltaCapacity_RunLiEss = SumRemainingCapacity_RunLiEss - SumRequiredCapacity_RunLiEss;  //总变化增量为负值
            SumPowerSet_RunLiEss = SumDeltaCapacity_RunLiEss / Time_Low;  //储能电池总充电功率=总待充电电量/谷段时长

            if (SumPowerSet_RunLiEss < SumMinPower_RunLiEss)
            {
                SumPowerSet_RunLiEss = SumMinPower_RunLiEss;
            }
            SystemOutput.DeltaP_Microgrid = SumPowerSet_RunLiEss + SumRealPower_Wind + SumRealPower_PV - SumRealPower_PCS;
        }
        else  //储能电池当前电量已足够，保持不充不放
        {
            SumPowerSet_RunLiEss = 0;
            SystemOutput.DeltaP_Microgrid = SumPowerSet_RunLiEss + SumRealPower_Wind + SumRealPower_PV - SumRealPower_PCS;
        }
    }
}

//风电总升有功功率调节能力评估函数
//评估某个风光储一体化单元中风电部分的总有功升功率调节能力，遍历单元内的所有风电机组，累加那些“状态正常、可准确细调、且当前有功功率未达上限”的机组的剩余上调功率。
double HaikouMicrogridEMSAlgorithm::WindIncreaseActivePowerCapacityCalculateFunction(Integration_Input IntegrationUnitInput)
{
    double Capacity_IncreaseActivePower_Wind_i = 0;  //第i个风光储单元中的风电升功率能力
    for (size_t j = 0; j < IntegrationUnitInput.Wind.size(); j = j + 1)
    {
        if (IntegrationUnitInput.Wind[j].Status == 0)  //该风电机组当前正常
        {
            if (IntegrationUnitInput.Wind[j].Flag_Adjust == 0)  //该风电机组功率不可准确细调
            {
                continue; // 机组不可准确细调，跳过，不贡献能力
            }
            else if (IntegrationUnitInput.Wind[j].Flag_Adjust == 1)  //该风电机组功率可以准确细调
            {
                if (IntegrationUnitInput.Wind[j].P_max > IntegrationUnitInput.Wind[j].P_real)
                {
                    Capacity_IncreaseActivePower_Wind_i = Capacity_IncreaseActivePower_Wind_i + IntegrationUnitInput.Wind[j].P_max - IntegrationUnitInput.Wind[j].P_real;
                }
                else
                {
                    continue; // 当前功率已达上限（P_max <= P_real），则该机组上调能力为0，跳过，不贡献能力
                }
            }
        }
        else if (IntegrationUnitInput.Wind[j].Status == 1)  //该风电机组当前故障
        {
            continue; // 机组故障，跳过，不贡献能力
        }
    }
    return Capacity_IncreaseActivePower_Wind_i;
}

//风电总降有功功率调节能力评估函数
//评估某个风光储一体化单元中风电部分的总有功降功率调节能力。遍历单元内的所有风电机组，累加降功率能力，返回值为负值或0，绝对值表示可下降的总功率。
//对于当前功率 > 最小功率的可调机组：下降能力 = 最小运行功率 - 当前功率；对于不可调机组：下降能力 = -当前功率(认为可通过停机降至0)
double HaikouMicrogridEMSAlgorithm::WindDecreaseActivePowerCapacityCalculateFunction(Integration_Input IntegrationUnitInput)
{
    double Capacity_DecreaseActivePower_Wind_i = 0;  //第i个风光储单元中的风电降功率能力
    for (size_t j = 0; j < IntegrationUnitInput.Wind.size(); j = j + 1)
    {
        if (IntegrationUnitInput.Wind[j].Status == 0)  //该风电机组当前正常
        {
            if (IntegrationUnitInput.Wind[j].Flag_Adjust == 0)  //该风电机组功率不可准确细调
            {
                Capacity_DecreaseActivePower_Wind_i = Capacity_DecreaseActivePower_Wind_i - IntegrationUnitInput.Wind[j].P_real;
            }
            else if (IntegrationUnitInput.Wind[j].Flag_Adjust == 1)  //该风电机组功率可以准确细调
            {
                if (IntegrationUnitInput.Wind[j].P_min < IntegrationUnitInput.Wind[j].P_real)
                {
                    Capacity_DecreaseActivePower_Wind_i = Capacity_DecreaseActivePower_Wind_i + IntegrationUnitInput.Wind[j].P_min - IntegrationUnitInput.Wind[j].P_real;
                }
                else
                {
                    continue; // 当前功率已达上限（P_min >= P_real），则该机组下调能力为0，跳过，不贡献能力
                }
            }
        }
        else if (IntegrationUnitInput.Wind[j].Status == 1)  //该风电机组当前故障
        {
            continue; // 机组故障，跳过，不贡献能力
        }
    }
    return Capacity_DecreaseActivePower_Wind_i;
}

//光伏总升有功功率调节能力评估函数
double HaikouMicrogridEMSAlgorithm::PVIncreaseActivePowerCapacityCalculateFunction(Configuration_Input Configuration, Integration_Input IntegrationUnitInput)
{
    // 如果DC/DC故障或者DC/DC不可准确细调，则增发功率能力为0
    if ((IntegrationUnitInput.DCDC_PV.FaultStatus_DCDC == 1) || (IntegrationUnitInput.DCDC_PV.Flag_Adjust == 0))
    {
        return 0.0;
    }

    //如果DC/DC无故障且DC/DC可以准确细调，则根据DC/DC运行状态进一步评估其增发功率能力
    double Capacity_IncreaseActivePower_PV_i = 0.0;    //第i个风光储单元中的光伏升功率能力
    const double StartTime_DCDC = 3.0;  // DC/DC模块启机时长=3s
    // 根据DC/DC的运行状态计算增发能力
    if (IntegrationUnitInput.DCDC_PV.RunStatus_DCDC == 0)  //该DC/DC当前处于停机状态
    {
        // 停机状态下，需要考虑启机时间
        if (Configuration.ControlCycle >= StartTime_DCDC)
        {
            Capacity_IncreaseActivePower_PV_i = IntegrationUnitInput.DCDC_PV.P_max;
        }
        else
        {
            Capacity_IncreaseActivePower_PV_i = Configuration.ControlCycle / StartTime_DCDC * IntegrationUnitInput.DCDC_PV.P_max;
        }
    }
    else if (IntegrationUnitInput.DCDC_PV.RunStatus_DCDC == 1)  //该DC/DC当前处于运行状态
    {
        // 运行状态下，增发能力为最大可发功率减去当前实际功率
        Capacity_IncreaseActivePower_PV_i = IntegrationUnitInput.DCDC_PV.P_max - IntegrationUnitInput.DCDC_PV.P_real;
    }

    // 确保升功率能力值为非负数（表示增加功率）
    if (Capacity_IncreaseActivePower_PV_i < 0)
    {
        Capacity_IncreaseActivePower_PV_i = 0.0;
    }

    return Capacity_IncreaseActivePower_PV_i;
}

//光伏总降有功功率调节能力评估函数
double HaikouMicrogridEMSAlgorithm::PVDecreaseActivePowerCapacityCalculateFunction(Integration_Input IntegrationUnitInput)
{
    // 如果DC/DC故障或者DC/DC停机，则减发功率能力为0
    if ((IntegrationUnitInput.DCDC_PV.FaultStatus_DCDC == 1) || (IntegrationUnitInput.DCDC_PV.RunStatus_DCDC == 0))
    {
        return 0.0;
    }

    // 如果DC/DC无故障且处于运行状态，则根据DC/DC可调性进一步评估其减发功率能力
    double Capacity_DecreaseActivePower_PV_i = 0;  //第i个风光储单元中的光伏降功率能力
    if (IntegrationUnitInput.DCDC_PV.Flag_Adjust == 0)  //该DC/DC不可准确细调
    {
        Capacity_DecreaseActivePower_PV_i = -IntegrationUnitInput.DCDC_PV.P_real;  //只能降到0（完全停机）
    }
    else if (IntegrationUnitInput.DCDC_PV.Flag_Adjust == 1)  //该DC/DC可以准确细调
    {
        Capacity_DecreaseActivePower_PV_i = IntegrationUnitInput.DCDC_PV.P_min - IntegrationUnitInput.DCDC_PV.P_real;  //可以降到最小运行功率
    }

    //确保降功率能力值为非正数（表示减少功率）
    if (Capacity_DecreaseActivePower_PV_i > 0)
    {
        Capacity_DecreaseActivePower_PV_i = 0;
    }

    return  Capacity_DecreaseActivePower_PV_i;
}

//储能电池总升有功功率调节能力评估函数
double HaikouMicrogridEMSAlgorithm::LiEssIncreaseActivePowerCapacityCalculateFunction(Configuration_Input Configuration, Integration_Input IntegrationUnitInput)
{
    double Capacity_IncreaseActivePower_LiEss_i = 0;  //第i个风光储单元中的储能电池升功率能力
    if (IntegrationUnitInput.LiEss.SOC_real <= Configuration.SOCmin_LiEss)  //微网EMS对电池禁放
    {
        Capacity_IncreaseActivePower_LiEss_i =  -IntegrationUnitInput.LiEss.P_real;  //升功率能力= -实时功率
    }
    else if (IntegrationUnitInput.LiEss.SOC_real > Configuration.SOCmin_LiEss)  //微网EMS未对电池禁放，可最大能力放电
    {
        Capacity_IncreaseActivePower_LiEss_i = IntegrationUnitInput.LiEss.P_max - IntegrationUnitInput.LiEss.P_real;  //升功率能力=最大可放电功率-实时功率
    }

    if ((IntegrationUnitInput.LiEss.Status == 3)||(IntegrationUnitInput.LiEss.Status == 4))   //BMS或储能EMS显示该储能电池当前处于待机或停机状态，不能放电，最大可放电功率=0
    {
        Capacity_IncreaseActivePower_LiEss_i = -IntegrationUnitInput.LiEss.P_real;  //升功率能力= -实时功率
    }

    /*
    if (IntegrationUnitInput.LiEss.Status == 1)  //BMS或储能EMS显示该储能电池当前处于禁充状态
    {
        Capacity_IncreaseActivePower_LiEss_i = IntegrationUnitInput.LiEss.P_max - IntegrationUnitInput.LiEss.P_real;  //升功率能力=最大可放电功率-实时功率
    }
    else if ((IntegrationUnitInput.LiEss.Status == 2)||(IntegrationUnitInput.LiEss.Status == 3)||(IntegrationUnitInput.LiEss.Status == 4))   //BMS或储能EMS显示该储能电池当前处于禁放、待机或停机状态，不能放电，最大可放电功率=0
    {
        Capacity_IncreaseActivePower_LiEss_i = -IntegrationUnitInput.LiEss.P_real;  //升功率能力= -实时功率
    }
    */

    if (Capacity_IncreaseActivePower_LiEss_i < 0)
    {
        Capacity_IncreaseActivePower_LiEss_i = 0;
    }
    return Capacity_IncreaseActivePower_LiEss_i;
}

//储能电池总降有功功率调节能力评估函数
double HaikouMicrogridEMSAlgorithm::LiEssDecreaseActivePowerCapacityCalculateFunction(Configuration_Input Configuration, Integration_Input IntegrationUnitInput)
{
    double Capacity_DecreaseActivePower_LiEss_i = 0;  //第i个风光储单元中的储能电池降功率能力
    if (IntegrationUnitInput.LiEss.SOC_real >= Configuration.SOCmax_LiEss)  //微网EMS对电池禁充
    {
        Capacity_DecreaseActivePower_LiEss_i = -IntegrationUnitInput.LiEss.P_real;  //降功率能力= -实时功率
    }
    else if (IntegrationUnitInput.LiEss.SOC_real < Configuration.SOCmax_LiEss)  //微网EMS未对电池禁充,可最大能力充电
    {
        Capacity_DecreaseActivePower_LiEss_i = IntegrationUnitInput.LiEss.P_min - IntegrationUnitInput.LiEss.P_real;  //降功率能力=最大可充电功率-实时功率
    }

    if ((IntegrationUnitInput.LiEss.Status == 3)||(IntegrationUnitInput.LiEss.Status == 4))  //BMS或储能EMS显示该储能电池当前处于待机或停机状态，不能充电，即最大可充电功率=0
    {
        Capacity_DecreaseActivePower_LiEss_i = -IntegrationUnitInput.LiEss.P_real;  //降功率能力= -实时功率
    }
    /*
    if ((IntegrationUnitInput.LiEss.Status == 1)||(IntegrationUnitInput.LiEss.Status == 3)||(IntegrationUnitInput.LiEss.Status == 4))  //BMS或储能EMS显示该储能电池当前处于禁充状态、待机或停机状态，不能充电，即最大可充电功率=0
    {
        Capacity_DecreaseActivePower_LiEss_i = -IntegrationUnitInput.LiEss.P_real;  //降功率能力= -实时功率
    }
    else if (IntegrationUnitInput.LiEss.Status == 2)  //BMS或储能EMS显示该储能电池当前处于禁放状态
    {
        Capacity_DecreaseActivePower_LiEss_i = IntegrationUnitInput.LiEss.P_min - IntegrationUnitInput.LiEss.P_real;  //降功率能力=最大可充电功率-实时功率
    }
    */

    if (Capacity_DecreaseActivePower_LiEss_i > 0)
    {
        Capacity_DecreaseActivePower_LiEss_i = 0;
    }
    return Capacity_DecreaseActivePower_LiEss_i;
}

//风光储单元的总升有功功率调节能力评估函数
double HaikouMicrogridEMSAlgorithm::PCSIncreaseActivePowerCapacityCalculateFunction(Configuration_Input Configuration, Integration_Input IntegrationUnitInput)
{
    double Capacity_IncreaseActivePower_PCS_i = 0;  //第i个风光储单元的总升功率能力
    for (size_t j = 0; j < IntegrationUnitInput.PCS.size(); j = j + 1)
    {
        if (IntegrationUnitInput.PCS[j].Type_PCS == 1)
        {
            if (IntegrationUnitInput.PCS[j].RunStatus_PCS == 0)  //该PCS当前处于停机状态
            {
                Capacity_IncreaseActivePower_PCS_i = 0;
            }
            else if (IntegrationUnitInput.PCS[j].RunStatus_PCS == 1)  //该PCS当前处于运行状态
            {
                if (IntegrationUnitInput.PCS[j].RunMode_PCS == 2)  //该PCS当前处于PQ运行模式
                {
                    double Capacity_IncreaseActivePower_Wind_i = WindIncreaseActivePowerCapacityCalculateFunction(IntegrationUnitInput);  //第i个风光储单元中风电的升功率能力
                    double Capacity_IncreaseActivePower_PV_i = PVIncreaseActivePowerCapacityCalculateFunction(Configuration, IntegrationUnitInput);  //第i个风光储单元中光伏的升功率能力
                    double Capacity_IncreaseActivePower_LiEss_i = LiEssIncreaseActivePowerCapacityCalculateFunction(Configuration, IntegrationUnitInput);  //第i个风光储单元中储能电池的升功率能力

                    Capacity_IncreaseActivePower_PCS_i = Capacity_IncreaseActivePower_Wind_i + Capacity_IncreaseActivePower_PV_i + Capacity_IncreaseActivePower_LiEss_i;  //单个风光储单元的总升功率能力=风光升功率能力+电池升功率能力
                    if (Capacity_IncreaseActivePower_PCS_i > (IntegrationUnitInput.PCS[j].P_max - IntegrationUnitInput.PCS[j].P_real))  //若超出PCS设备本身的最大升功率能力
                    {
                        if (IntegrationUnitInput.PCS[j].P_max > IntegrationUnitInput.PCS[j].P_real)
                        {
                            Capacity_IncreaseActivePower_PCS_i = IntegrationUnitInput.PCS[j].P_max - IntegrationUnitInput.PCS[j].P_real;  //单个风光储单元的总升功率能力=PCS的最大升功率能力
                        }
                        else
                        {
                            Capacity_IncreaseActivePower_PCS_i = 0;  //单个风光储单元的总升功率能力=0
                        }
                    }
                }
                else
                {
                    Capacity_IncreaseActivePower_PCS_i = 0;  //单个风光储单元的总升功率能力=0
                }
            }
        }
        else
        {
            continue;
        }
    }
    return Capacity_IncreaseActivePower_PCS_i;
}

//风光储单元的总降有功功率调节能力评估函数
double HaikouMicrogridEMSAlgorithm::PCSDecreaseActivePowerCapacityCalculateFunction(Configuration_Input Configuration, Integration_Input IntegrationUnitInput)
{
    double Capacity_DecreaseActivePower_PCS_i = 0;  //第i个风光储单元的总降功率能力
    for (size_t j = 0; j < IntegrationUnitInput.PCS.size(); j = j + 1)
    {
        if (IntegrationUnitInput.PCS[j].Type_PCS == 1)
        {
            if (IntegrationUnitInput.PCS[j].RunStatus_PCS == 0)  //该PCS当前处于停机状态
            {
                Capacity_DecreaseActivePower_PCS_i = 0;
            }
            else if (IntegrationUnitInput.PCS[j].RunStatus_PCS == 1)  //该PCS当前处于运行状态
            {
                if (IntegrationUnitInput.PCS[j].RunMode_PCS == 2)  //该PCS当前处于PQ运行模式
                {
                    if ((IntegrationUnitInput.LiEss.SOC_real >= Configuration.SOCmax_LiEss) && (IntegrationUnitInput.PCS[j].P_real <= 0.01*IntegrationUnitInput.PCS[j].Pn))
                    {
                        Capacity_DecreaseActivePower_PCS_i = 0;
                        continue;
                    }
                    double Capacity_DecreaseActivePower_Wind_i = WindDecreaseActivePowerCapacityCalculateFunction(IntegrationUnitInput);  //第i个风光储单元中风电的降功率能力
                    double Capacity_DecreaseActivePower_PV_i = PVDecreaseActivePowerCapacityCalculateFunction(IntegrationUnitInput);  //第i个风光储单元中光伏的降功率能力
                    double Capacity_DecreaseActivePower_LiEss_i = LiEssDecreaseActivePowerCapacityCalculateFunction(Configuration, IntegrationUnitInput);  //第i个风光储单元中储能电池的降功率能力
                    Capacity_DecreaseActivePower_PCS_i = Capacity_DecreaseActivePower_Wind_i + Capacity_DecreaseActivePower_PV_i + Capacity_DecreaseActivePower_LiEss_i;  //单个风光储单元的总降功率能力=风光降功率能力+电池降功率能力

                    if (Capacity_DecreaseActivePower_PCS_i < (IntegrationUnitInput.PCS[j].P_min - IntegrationUnitInput.PCS[j].P_real))  //若超出PCS设备本身的最大降功率能力
                    {
                        if (IntegrationUnitInput.PCS[j].P_min < IntegrationUnitInput.PCS[j].P_real)
                        {
                            Capacity_DecreaseActivePower_PCS_i = IntegrationUnitInput.PCS[j].P_min - IntegrationUnitInput.PCS[j].P_real;  //单个风光储单元的总降功率能力=PCS的最大降功率能力
                        }
                        else
                        {
                            Capacity_DecreaseActivePower_PCS_i = 0;  //单个风光储单元的总降功率能力=0
                        }
                    }
                }
                else
                {
                    Capacity_DecreaseActivePower_PCS_i = 0;  //单个风光储单元的总降功率能力=0
                }
            }
        }
        else
        {
            continue;
        }
    }
    return Capacity_DecreaseActivePower_PCS_i;
}

//微网升有功功率分配控制函数
void HaikouMicrogridEMSAlgorithm::IncreaseActivePowerAllocateControlFunction(Configuration_Input Configuration, vector<Integration_Input> IntegrationInput, System_Output& SystemOutput, vector<Integration_Output>& IntegrationOutput)
{
    // 1. 计算微电网总升功率能力
    double SumCapacity_IncreaseActivePower_Microgrid = 0;  //微电网的总升功率能力
    vector<double> UnitIncreaseCapacities(IntegrationInput.size()); // 存储每个单元的能力
    double SumSOC_LiEss = 0;  //总SOC
    for (size_t i = 0; i < IntegrationInput.size(); i = i + 1)
    {
        UnitIncreaseCapacities[i] = PCSIncreaseActivePowerCapacityCalculateFunction(Configuration, IntegrationInput[i]);  //单个风光储单元的总升功率能力
        SumCapacity_IncreaseActivePower_Microgrid = SumCapacity_IncreaseActivePower_Microgrid + UnitIncreaseCapacities[i];
        SumSOC_LiEss = SumSOC_LiEss + IntegrationInput[i].LiEss.SOC_real;
    }

    // 2. 检查是否超出微网调节能力
    if (SystemOutput.DeltaP_Microgrid > SumCapacity_IncreaseActivePower_Microgrid)  //微网总有功目标调节增量>微网总升功率能力
    {
        SystemOutput.Flag_ExceedMicrogridCapability = 1;  //超出微网增发功率能力
        SystemOutput.DeltaP_Microgrid = SumCapacity_IncreaseActivePower_Microgrid;
    }
    else
    {
        SystemOutput.Flag_ExceedMicrogridCapability = 0;  //未超出微网调节能力
    }

    // 3. 创建IntegrationInput的索引数组并按照SOC从大到小排序
    vector<size_t> IntegrationUnitIndices(IntegrationInput.size());  //创建风光储一体化单元索引数组，初始大小与IntegrationInput相同
    for (size_t i = 0; i < IntegrationInput.size(); i++)  //用索引值填充数组：[0, 1, 2, ..., IntegrationInput.size()-1]
    {
        IntegrationUnitIndices[i] = i;
    }

    // 使用sort函数对风光储一体化单元的索引（IntegrationUnitIndices）进行排序，而不是直接对单元数组中的原始数据（IntegrationInput）排序，进行索引排序可避免在排序过程中移动大量数据（特别是当数组元素是较大的结构体时，需复制整个结构体数组，内存开销大），从而提高效率。
    // C++ STL中的sort函数，用于对IntegrationUnitIndices这个索引数组的所有元素进行排序，使用时需包含头文件#include<algorithm>。begin()和end()确定了排序范围；lambda表达式（即匿名函数）确定了排序的比较规则（如按照SOC从大到小），其中：
    // [&]：捕获列表，表示以引用方式捕获所有外部变量，这样在lambda函数体内可使用当前作用域内的所有变量（例如IntegrationInput）；(size_t a, size_t b)：两个参数，分别表示IntegrationUnitIndices数组中的两个元素；
    // 函数体：{return IntegrationInput[a].LiEss.SOC_real > IntegrationInput[b].LiEss.SOC_real;}。比较规则的具体实现,比较的是通过索引a和b访问的IntegrationInput数组中两个风光储一体化单元中储能电池的SOC值。如果a对应的SOC值大于b对应的SOC值，则返回true，表示a应排在b的前面，保持现有顺序；反之则返回false，a与b的顺序交换。
    sort(IntegrationUnitIndices.begin(), IntegrationUnitIndices.end(), [&](size_t a, size_t b) {return IntegrationInput[a].LiEss.SOC_real > IntegrationInput[b].LiEss.SOC_real; });

    // 4. 1）按 SOC 比例初次分配：每个单元按其 SOC 占 SOC 总和的比例分配所需调节增量，但不超过其最大升功率能力。
    //    2）再按 SOC 从大到小补充分配：若初次分配后仍有剩余增量，则按 SOC 从高到低对仍有调节能力的单元进行补充分配，直至分配完毕或能力耗尽。
    //IntegrationOutput.resize(IntegrationInput.size());  //初始化输出向量，确保大小正确
    if(IntegrationOutput.size()!=IntegrationInput.size())
    {
        IntegrationOutput.resize(IntegrationInput.size());  //初始化输出向量，确保大小正确
    }

    double AllocatedDeltaP_Microgrid = SystemOutput.DeltaP_Microgrid;  //待分配的微电网总有功目标调节增量
    double SumRatio_deltaP_set = 0;  //按 SOC 比例初次分配，已分配的总增量
    vector<bool> isFull(IntegrationInput.size(), false);             // 标记是否已达能力上限
    for (size_t i = 0; i < IntegrationInput.size(); i++)
    {
        for (size_t j = 0; j < IntegrationInput[i].PCS.size(); j = j + 1)
        {
            if (IntegrationInput[i].PCS[j].Type_PCS == 1)  //该PCS为用于连接负荷与风光储的双向变流器型PCS
            {
                IntegrationOutput[i].PCS[j].ID_PCS = IntegrationInput[i].PCS[j].ID_PCS;
                double ratio = IntegrationInput[i].LiEss.SOC_real / SumSOC_LiEss;
                double ratio_deltaP_set_i = ratio * AllocatedDeltaP_Microgrid;
                if(ratio_deltaP_set_i > UnitIncreaseCapacities[i])
                {
                    IntegrationOutput[i].PCS[j].deltaP_set = UnitIncreaseCapacities[i];
                    isFull[i] = true;
                }
                else
                {
                    IntegrationOutput[i].PCS[j].deltaP_set = ratio_deltaP_set_i;
                }
            }
            SumRatio_deltaP_set = SumRatio_deltaP_set + IntegrationOutput[i].PCS[j].deltaP_set;
        }
    }

    AllocatedDeltaP_Microgrid = AllocatedDeltaP_Microgrid - SumRatio_deltaP_set;  //更新待分配的微电网总有功目标调节增量，即剩余待分配量
    if (AllocatedDeltaP_Microgrid > 0)  //还未完成分配
    {
        for (size_t i = 0; i < IntegrationUnitIndices.size(); i++)
        {
            size_t i_selected = IntegrationUnitIndices[i];
            if (AllocatedDeltaP_Microgrid > 0)  //还未完成分配
            {
                if(isFull[i_selected])  //已达能力上限
                {
                    continue;
                }
                else
                {
                    for (size_t j = 0; j < IntegrationInput[i_selected].PCS.size(); j = j + 1)
                    {
                        if (IntegrationInput[i_selected].PCS[j].Type_PCS == 1)  //该PCS为用于连接负荷与风光储的双向变流器型PCS
                        {
                            IntegrationOutput[i_selected].PCS[j].ID_PCS = IntegrationInput[i_selected].PCS[j].ID_PCS;

                            double capacityLeft_i = UnitIncreaseCapacities[i_selected] - IntegrationOutput[i_selected].PCS[j].deltaP_set;  //第i个风光储单元剩余的调节能力
                            if(capacityLeft_i < AllocatedDeltaP_Microgrid)
                            {
                                IntegrationOutput[i_selected].PCS[j].deltaP_set = UnitIncreaseCapacities[i_selected];  //第i个风光储单元尽最大能力调节，所分配的调节增量=最大升功率能力
                                AllocatedDeltaP_Microgrid = AllocatedDeltaP_Microgrid - capacityLeft_i;
                            }
                            else
                            {
                                IntegrationOutput[i_selected].PCS[j].deltaP_set = IntegrationOutput[i_selected].PCS[j].deltaP_set + AllocatedDeltaP_Microgrid;
                                AllocatedDeltaP_Microgrid = 0;
                                break;
                            }
                        }
                    }
                }
            }
            else  //已经完成分配
            {
                break;
            }
        }
    }

    for (size_t i = 0; i < IntegrationInput.size(); i++)
    {
        for (size_t j = 0; j < IntegrationInput[i].PCS.size(); j = j + 1)
        {
            if (IntegrationInput[i].PCS[j].Type_PCS == 1)  //该PCS为用于连接负荷与风光储的双向变流器型PCS
            {
                IntegrationOutput[i].PCS[j].ID_PCS = IntegrationInput[i].PCS[j].ID_PCS;
                PCSActivePowerSetCalculateFunction(IntegrationInput[i].PCS[j], IntegrationOutput[i].PCS[j]);  //第i个风光储单元有功设定值的计算与修正
            }
        }
    }
}

//微网降有功功率分配控制函数
void HaikouMicrogridEMSAlgorithm::DecreaseActivePowerAllocateControlFunction(Configuration_Input Configuration, vector<Integration_Input> IntegrationInput, System_Output& SystemOutput, vector<Integration_Output>& IntegrationOutput)
{
    // 1. 计算微电网总降功率能力
    double SumCapacity_DecreaseActivePower_Microgrid = 0;  //微电网的总降功率能力
    vector<double> UnitDecreaseCapacities(IntegrationInput.size()); // 存储每个单元的降功率能力
    double SumSOC_LiEss = 0;  //总SOC
    for (size_t i = 0; i < IntegrationInput.size(); i = i + 1)
    {
        UnitDecreaseCapacities[i] = PCSDecreaseActivePowerCapacityCalculateFunction(Configuration, IntegrationInput[i]);  //单个风光储单元的降功率能力
        SumCapacity_DecreaseActivePower_Microgrid = SumCapacity_DecreaseActivePower_Microgrid + UnitDecreaseCapacities[i];
        SumSOC_LiEss = SumSOC_LiEss + IntegrationInput[i].LiEss.SOC_real;
    }

    // 2. 检查是否超出微网调节能力
    if (SystemOutput.DeltaP_Microgrid < SumCapacity_DecreaseActivePower_Microgrid)  //微网总有功目标调节增量<微网总降功率能力
    {
        SystemOutput.Flag_ExceedMicrogridCapability = 2;  //超出微网减发功率能力 
        SystemOutput.DeltaP_Microgrid = SumCapacity_DecreaseActivePower_Microgrid;
    }
    else
    {
        SystemOutput.Flag_ExceedMicrogridCapability = 0;  //未超出微网调节能力
    }

    // 3. 创建IntegrationInput的索引数组并按照SOC从小到大排序
    vector<size_t> IntegrationUnitIndices(IntegrationInput.size());  //创建风光储一体化单元索引数组，初始大小与IntegrationInput相同
    for (size_t i = 0; i < IntegrationInput.size(); i++)  //用索引值填充数组：[0, 1, 2, ..., IntegrationInput.size()-1]
    {
        IntegrationUnitIndices[i] = i;
    }

    // 使用sort函数对风光储一体化单元的索引（IntegrationUnitIndices）进行排序，而不是直接对单元数组中的原始数据（IntegrationInput）排序，进行索引排序可避免在排序过程中移动大量数据（特别是当数组元素是较大的结构体时，需复制整个结构体数组，内存开销大），从而提高效率。
    // C++ STL中的sort函数，用于对IntegrationUnitIndices这个索引数组的所有元素进行排序，使用时需包含头文件#include<algorithm>。begin()和end()确定了排序范围；lambda表达式（即匿名函数）确定了排序的比较规则（如按照SOC从小到大），其中：
    // [&]：捕获列表，表示以引用方式捕获所有外部变量，这样在lambda函数体内可使用当前作用域内的所有变量（例如IntegrationInput）；(size_t a, size_t b)：两个参数，分别表示IntegrationUnitIndices数组中的两个元素；
    // 函数体：{return IntegrationInput[a].LiEss.SOC_real < IntegrationInput[b].LiEss.SOC_real;}。比较规则的具体实现,比较的是通过索引a和b访问的IntegrationInput数组中两个风光储一体化单元中储能电池的SOC值。如果a对应的SOC值小于b对应的SOC值，则返回true，表示a应排在b的前面，保持现有顺序；反之则返回false，a与b的顺序交换。
    sort(IntegrationUnitIndices.begin(), IntegrationUnitIndices.end(), [&](size_t a, size_t b) {return IntegrationInput[a].LiEss.SOC_real < IntegrationInput[b].LiEss.SOC_real; });

    // 4. 1）按 SOC 比例初次分配：每个单元按其 SOC 占 SOC 总和的比例分配所需调节增量，但不超过其最大升功率能力。
    //    2）再按 SOC 从大到小补充分配：若初次分配后仍有剩余增量，则按 SOC 从高到低对仍有调节能力的单元进行补充分配，直至分配完毕或能力耗尽。
    //IntegrationOutput.resize(IntegrationInput.size());  //初始化输出向量，确保大小正确
    if(IntegrationOutput.size()!=IntegrationInput.size())
    {
        IntegrationOutput.resize(IntegrationInput.size());  //初始化输出向量，确保大小正确
    }

    double AllocatedDeltaP_Microgrid = SystemOutput.DeltaP_Microgrid;  //待分配的微电网总有功目标调节增量
    double SumRatio_deltaP_set = 0;  //按 SOC 比例初次分配，已分配的总增量
    vector<bool> isFull(IntegrationInput.size(), false);             // 标记是否已达能力上限
    for (size_t i = 0; i < IntegrationInput.size(); i++)
    {
        for (size_t j = 0; j < IntegrationInput[i].PCS.size(); j = j + 1)
        {
            if (IntegrationInput[i].PCS[j].Type_PCS == 1)  //该PCS为用于连接负荷与风光储的双向变流器型PCS
            {
                IntegrationOutput[i].PCS[j].ID_PCS = IntegrationInput[i].PCS[j].ID_PCS;
                double ratio = (100-IntegrationInput[i].LiEss.SOC_real) / SumSOC_LiEss;
                double ratio_deltaP_set_i = ratio * AllocatedDeltaP_Microgrid;
                if(ratio_deltaP_set_i < UnitDecreaseCapacities[i])
                {
                    IntegrationOutput[i].PCS[j].deltaP_set = UnitDecreaseCapacities[i];
                    isFull[i] = true;
                }
                else
                {
                    IntegrationOutput[i].PCS[j].deltaP_set = ratio_deltaP_set_i;
                }
            }
            SumRatio_deltaP_set = SumRatio_deltaP_set + IntegrationOutput[i].PCS[j].deltaP_set;
        }
    }

    AllocatedDeltaP_Microgrid = AllocatedDeltaP_Microgrid - SumRatio_deltaP_set;  //更新待分配的微电网总有功目标调节增量，即剩余待分配量
    if (AllocatedDeltaP_Microgrid < 0)  //还未完成分配
    {
        for (size_t i = 0; i < IntegrationUnitIndices.size(); i++)
        {
            size_t i_selected = IntegrationUnitIndices[i];
            if (AllocatedDeltaP_Microgrid < 0)  //还未完成分配
            {
                if(isFull[i_selected])  //已达能力上限
                {
                    continue;
                }
                else
                {
                    for (size_t j = 0; j < IntegrationInput[i_selected].PCS.size(); j = j + 1)
                    {
                        if (IntegrationInput[i_selected].PCS[j].Type_PCS == 1)  //该PCS为用于连接负荷与风光储的双向变流器型PCS
                        {
                            IntegrationOutput[i_selected].PCS[j].ID_PCS = IntegrationInput[i_selected].PCS[j].ID_PCS;

                            double capacityLeft_i = UnitDecreaseCapacities[i_selected] - IntegrationOutput[i_selected].PCS[j].deltaP_set;  //第i个风光储单元剩余的调节能力
                            if(capacityLeft_i > AllocatedDeltaP_Microgrid)  //第i个风光储单元剩余的调节能力不满足剩余待分配量
                            {
                                IntegrationOutput[i_selected].PCS[j].deltaP_set = UnitDecreaseCapacities[i_selected];  //第i个风光储单元尽最大能力调节，所分配的调节增量=最大降功率能力
                                AllocatedDeltaP_Microgrid = AllocatedDeltaP_Microgrid - capacityLeft_i;
                            }
                            else
                            {
                                IntegrationOutput[i_selected].PCS[j].deltaP_set = IntegrationOutput[i_selected].PCS[j].deltaP_set + AllocatedDeltaP_Microgrid;
                                AllocatedDeltaP_Microgrid = 0;
                                break;
                            }
                        }
                    }
                }
            }
            else  //已经完成分配
            {
                break;
            }
        }
    }

    for (size_t i = 0; i < IntegrationInput.size(); i++)
    {
        for (size_t j = 0; j < IntegrationInput[i].PCS.size(); j = j + 1)
        {
            if (IntegrationInput[i].PCS[j].Type_PCS == 1)  //该PCS为用于连接负荷与风光储的双向变流器型PCS
            {
                IntegrationOutput[i].PCS[j].ID_PCS = IntegrationInput[i].PCS[j].ID_PCS;
                PCSActivePowerSetCalculateFunction(IntegrationInput[i].PCS[j], IntegrationOutput[i].PCS[j]);  //第i个风光储单元有功设定值的计算与修正
            }
        }
    }
}

//风光储一体化单元（单个PCS）有功功率设定值计算与修正函数
void HaikouMicrogridEMSAlgorithm::PCSActivePowerSetCalculateFunction(PCS_Input PCSInput, PCS_Output& PCSOutput)
{
    PCSOutput.P_set=0;

    if (PCSInput.Type_PCS == 1)  //该PCS为用于连接负荷与风光储的双向变流器型PCS
    {
        if (fabs(PCSInput.P_set_1 - PCSInput.P_real) <= 0.02 * PCSInput.Pn)
        {
            PCSOutput.P_set = PCSInput.P_set_1 + PCSOutput.deltaP_set;
        }
        else
        {
            PCSOutput.P_set = PCSInput.P_real + PCSOutput.deltaP_set;
        }

        if (PCSOutput.P_set > PCSInput.P_max)
        {
            PCSOutput.P_set = PCSInput.P_max;
        }
        else if (PCSOutput.P_set < PCSInput.P_min)
        {
            PCSOutput.P_set = PCSInput.P_min;
        }
    }
}

//微网并网无功功率控制函数
void HaikouMicrogridEMSAlgorithm::GridConnectedReactivePowerControlFunction(Configuration_Input Configuration, ExternalPowerGrid_Input ExternalPowerGrid, vector<Integration_Input> IntegrationInput, System_Output& SystemOutput, vector<Integration_Output>& IntegrationOutput)
{
    DeltaQCalculationFunction(Configuration, ExternalPowerGrid, SystemOutput);  //微网总无功目标调节增量计算，得到当前周期的待控制偏差量

    //增加无功闭环修正控制
    double ControlDeviationQ_1 = ExternalPowerGrid.Q_real_pcc - ExternalPowerGrid.Q_previousone_pcc - ExternalPowerGrid.DeltaQ_Microgrid_1;  //上一周期控制误差
    SystemOutput.DeltaQ_Microgrid = Configuration.Kp_ReactivePowerControl * (SystemOutput.DeltaQ_Microgrid - ControlDeviationQ_1) + Configuration.Ki_ReactivePowerControl * SystemOutput.DeltaQ_Microgrid * Configuration.ControlCycle;
    //double maxstep_DeltaQ = 20.0; // 无功增量限幅
    //double minstep_DeltaQ = -20.0;

    if (SystemOutput.DeltaQ_Microgrid > Configuration.DeadZone_IncreaseReactivePower)	//微网总有功目标调节增量>升无功功率控制死区时，进行微网无功升功率控制
    {
        //微网升无功功率分配控制与无功设定值计算函数
        IncreaseReactivePowerAllocateControlFunction(Configuration, IntegrationInput, SystemOutput, IntegrationOutput);
    }

    else if (SystemOutput.DeltaQ_Microgrid < Configuration.DeadZone_DecreaseReactivePower)  //微网总无功目标调节增量<降无功功率控制死区时，进行微网无功降功率控制
    {
        //微网降无功功率分配控制与无功设定值计算函数
        DecreaseReactivePowerAllocateControlFunction(Configuration, IntegrationInput, SystemOutput, IntegrationOutput);
    }
    else if ((SystemOutput.DeltaQ_Microgrid <= Configuration.DeadZone_IncreaseReactivePower) && (SystemOutput.DeltaQ_Microgrid >= Configuration.DeadZone_DecreaseReactivePower))  //在升降无功功率控制死区范围内，微网无功不增不减，维持不变
    {
        SystemOutput.DeltaQ_Microgrid = 0;
        SystemOutput.Flag_ExceedQCapability = 0;
        for (size_t i = 0;i < IntegrationInput.size();i = i + 1)
        {
            for (size_t j = 0; j < IntegrationInput[i].PCS.size(); j = j + 1)
            {
                if (IntegrationInput[i].PCS[j].Type_PCS == 1)  //该PCS为用于连接负荷与风光储的双向变流器型PCS
                {
                    IntegrationOutput[i].PCS[j].ID_PCS = IntegrationInput[i].PCS[j].ID_PCS;
                    IntegrationOutput[i].PCS[j].deltaQ_set = 0;
                    PCSReactivePowerSetCalculateFunction(IntegrationInput[i].PCS[j], IntegrationOutput[i].PCS[j], 0.03 * IntegrationInput[i].PCS[j].Pn * Configuration.ControlCycle);  //第i个风光储单元无功设定值的计算与修正，更新IntegrationOutput[i].PCS[j].Q_set
                }
                else
                {
                    continue;
                }
            }
        }
    }

    for (size_t i = 0;i < IntegrationInput.size();i = i + 1)
    {
        for (size_t j = 0; j < IntegrationInput[i].PCS.size(); j = j + 1)
        {
            if (IntegrationInput[i].PCS[j].Type_PCS == 1)  //该PCS为用于连接负荷与风光储的双向变流器型PCS
            {
                 SystemOutput.Qset_Microgrid = SystemOutput.Qset_Microgrid + IntegrationOutput[i].PCS[j].Q_set;
            }
        }
    }
}

//微电网并网运行总无功目标调节增量计算函数
void HaikouMicrogridEMSAlgorithm::DeltaQCalculationFunction(Configuration_Input Configuration, ExternalPowerGrid_Input ExternalPowerGrid, System_Output& SystemOutput)
{
    //一、根据交换无功功率大小分工况进行控制：超上限控制、超下限控制、保持不变。（交换无功功率<0时，电网提供感性无功；反之电网提供容性无功）
    //1）实时交换无功功率介于上限值与下限值之间，保持不变。
    if ((ExternalPowerGrid.Q_real_pcc >= Configuration.Qmin_pcc) && (ExternalPowerGrid.Q_real_pcc <= Configuration.Qmax_pcc))
    {
        SystemOutput.DeltaQ_Microgrid = 0;  //微网输出无功功率保持不变，此时DeltaQ_Microgrid>0
    }
    //2）实时交换无功功率大于上限值，进行超上限控制。
    else if (ExternalPowerGrid.Q_real_pcc > Configuration.Qmax_pcc)
    {
        SystemOutput.DeltaQ_Microgrid = -(Configuration.Qmax_pcc - ExternalPowerGrid.Q_real_pcc);  //减小微网输出无功功率(如：Q_real_pcc=4，Qmin_pcc=0)，此时DeltaQ_Microgrid>0
    }
    //3）实时交换功率小于下送功率预警值，进行超下限控制。
    else if (ExternalPowerGrid.Q_real_pcc < Configuration.Qmin_pcc)
    {
        SystemOutput.DeltaQ_Microgrid = -(Configuration.Qmin_pcc - ExternalPowerGrid.Q_real_pcc);  //减少电网提供的感性无功(如：Q_real_pcc=-10，Qmin_pcc=-2)，增加微网输出的感性无功(PCS输出感性无功时为负值)，此时DeltaQ_Microgrid=<0
    }
}

//微网升无功功率分配控制函数
void HaikouMicrogridEMSAlgorithm::IncreaseReactivePowerAllocateControlFunction(Configuration_Input Configuration, vector<Integration_Input> IntegrationInput, System_Output& SystemOutput, vector<Integration_Output>& IntegrationOutput)
{
    // 1. 计算微电网总升无功率能力
    double SumCapacity_IncreaseReactivePower_Microgrid = 0;  //微电网的总升无功功率能力
    vector<double> UnitIncreaseQCapacities(IntegrationInput.size()); // 存储每个单元的无功能力
    for (size_t i = 0; i < IntegrationInput.size(); i = i + 1)
    {
        UnitIncreaseQCapacities[i] = PCSIncreaseReactivePowerCapacityCalculateFunction(IntegrationInput[i], IntegrationOutput[i]);  //单个风光储单元的总升无功功率能力
        SumCapacity_IncreaseReactivePower_Microgrid = SumCapacity_IncreaseReactivePower_Microgrid + UnitIncreaseQCapacities[i];
    }

    // 2. 检查是否超出微网无功调节能力
    if (SystemOutput.DeltaQ_Microgrid > SumCapacity_IncreaseReactivePower_Microgrid)  //微网总无功目标调节增量>微网总升无功功率能力
    {
        SystemOutput.Flag_ExceedQCapability = 1;  //超出微网增发无功功率能力
        SystemOutput.DeltaQ_Microgrid = SumCapacity_IncreaseReactivePower_Microgrid;
    }
    else
    {
        SystemOutput.Flag_ExceedQCapability = 0;  //未超出微网调节能力
    }

    // 3. 按无功调节能力的比例分配：每个单元按其无功占 无功能力总和的比例分配所需调节增量，但不超过其最大升无功功率能力。
    if(IntegrationOutput.size()!=IntegrationInput.size())
    {
        IntegrationOutput.resize(IntegrationInput.size());  //初始化输出向量，确保大小正确
    }

    double AllocatedDeltaQ_Microgrid = SystemOutput.DeltaQ_Microgrid;  //待分配的微电网总有功目标调节增量
    for (size_t i = 0; i < IntegrationInput.size(); i++)
    {
        for (size_t j = 0; j < IntegrationInput[i].PCS.size(); j = j + 1)
        {
            if (IntegrationInput[i].PCS[j].Type_PCS == 1)  //该PCS为用于连接负荷与风光储的双向变流器型PCS
            {
                IntegrationOutput[i].PCS[j].ID_PCS = IntegrationInput[i].PCS[j].ID_PCS;
                double ratio = UnitIncreaseQCapacities[i] / SumCapacity_IncreaseReactivePower_Microgrid;
                double ratio_deltaQ_set_i = ratio * AllocatedDeltaQ_Microgrid;
                if(ratio_deltaQ_set_i > UnitIncreaseQCapacities[i])
                {
                    IntegrationOutput[i].PCS[j].deltaQ_set = UnitIncreaseQCapacities[i];
                    SystemOutput.DeltaQ_Microgrid = SystemOutput.DeltaQ_Microgrid - (ratio_deltaQ_set_i - UnitIncreaseQCapacities[i]);
                }
                else
                {
                    IntegrationOutput[i].PCS[j].deltaQ_set = ratio_deltaQ_set_i;
                }

                double ramp_limit = 0.03 * IntegrationInput[i].PCS[j].Pn;   // 斜坡速率限制（单位：kvar/s），建议设为额定无功的5%/s
                // 计算本周期允许的最大无功变化量 = 斜坡限制 * 控制周期
                double max_step = ramp_limit * Configuration.ControlCycle;
                if(IntegrationOutput[i].PCS[j].deltaQ_set > max_step)
                {
                    IntegrationOutput[i].PCS[j].deltaQ_set = max_step;
                    SystemOutput.DeltaQ_Microgrid = SystemOutput.DeltaQ_Microgrid - (IntegrationOutput[i].PCS[j].deltaQ_set - max_step);
                }

                PCSReactivePowerSetCalculateFunction(IntegrationInput[i].PCS[j], IntegrationOutput[i].PCS[j], max_step);  //第i个风光储单元无功设定值的计算与修正
            }
        }
    }
}

//微网降无功功率分配控制函数
void HaikouMicrogridEMSAlgorithm::DecreaseReactivePowerAllocateControlFunction(Configuration_Input Configuration, vector<Integration_Input> IntegrationInput, System_Output& SystemOutput, vector<Integration_Output>& IntegrationOutput)
{
    // 1. 计算微电网总降无功率能力
    double SumCapacity_DecreaseReactivePower_Microgrid = 0;  //微电网的总降无功功率能力
    vector<double> UnitDecreaseQCapacities(IntegrationInput.size()); // 存储每个单元的无功能力
    for (size_t i = 0; i < IntegrationInput.size(); i = i + 1)
    {
        UnitDecreaseQCapacities[i] = PCSDecreaseReactivePowerCapacityCalculateFunction(IntegrationInput[i], IntegrationOutput[i]);  //单个风光储单元的总降无功功率能力
        SumCapacity_DecreaseReactivePower_Microgrid = SumCapacity_DecreaseReactivePower_Microgrid + UnitDecreaseQCapacities[i];
    }

    // 2. 检查是否超出微网无功调节能力
    if (SystemOutput.DeltaQ_Microgrid < SumCapacity_DecreaseReactivePower_Microgrid)  //微网总无功目标调节增量<微网总降无功功率能力
    {
        SystemOutput.Flag_ExceedQCapability = 1;  //超出微网减发无功功率能力
        SystemOutput.DeltaQ_Microgrid = SumCapacity_DecreaseReactivePower_Microgrid;
    }
    else
    {
        SystemOutput.Flag_ExceedQCapability = 0;  //未超出微网无功调节能力
    }

    // 3. 按无功调节能力的比例分配：每个单元按其无功占 无功能力总和的比例分配所需调节增量，但不超过其最大降无功功率能力。
    if(IntegrationOutput.size()!=IntegrationInput.size())
    {
        IntegrationOutput.resize(IntegrationInput.size());  //初始化输出向量，确保大小正确
    }

    double AllocatedDeltaQ_Microgrid = SystemOutput.DeltaQ_Microgrid;  //待分配的微电网总有功目标调节增量
    for (size_t i = 0; i < IntegrationInput.size(); i++)
    {
        for (size_t j = 0; j < IntegrationInput[i].PCS.size(); j = j + 1)
        {
            if (IntegrationInput[i].PCS[j].Type_PCS == 1)  //该PCS为用于连接负荷与风光储的双向变流器型PCS
            {
                IntegrationOutput[i].PCS[j].ID_PCS = IntegrationInput[i].PCS[j].ID_PCS;
                double ratio = UnitDecreaseQCapacities[i] / SumCapacity_DecreaseReactivePower_Microgrid;
                double ratio_deltaQ_set_i = ratio * AllocatedDeltaQ_Microgrid;
                if(ratio_deltaQ_set_i < UnitDecreaseQCapacities[i])
                {
                    IntegrationOutput[i].PCS[j].deltaQ_set = UnitDecreaseQCapacities[i];
                    SystemOutput.DeltaQ_Microgrid = SystemOutput.DeltaQ_Microgrid - (ratio_deltaQ_set_i - UnitDecreaseQCapacities[i]);
                }
                else
                {
                    IntegrationOutput[i].PCS[j].deltaQ_set = ratio_deltaQ_set_i;
                }

                double ramp_limit = -0.03 * IntegrationInput[i].PCS[j].Pn;   // 斜坡速率限制（单位：kvar/s），建议设为额定无功的5%/s
                // 计算本周期允许的最大无功变化量 = 斜坡限制 * 控制周期
                double min_step = ramp_limit * Configuration.ControlCycle;
                if(IntegrationOutput[i].PCS[j].deltaQ_set < min_step)
                {
                    IntegrationOutput[i].PCS[j].deltaQ_set = min_step;
                    SystemOutput.DeltaQ_Microgrid = SystemOutput.DeltaQ_Microgrid - (IntegrationOutput[i].PCS[j].deltaQ_set - min_step);
                }

                PCSReactivePowerSetCalculateFunction(IntegrationInput[i].PCS[j], IntegrationOutput[i].PCS[j], min_step);  //第i个风光储单元无功设定值的计算与修正
            }
        }
    }
}

//风光储单元的总升无功功率调节能力评估函数
double HaikouMicrogridEMSAlgorithm::PCSIncreaseReactivePowerCapacityCalculateFunction(Integration_Input IntegrationUnitInput, Integration_Output IntegrationUnitOutput)
{
    double Capacity_IncreaseReactivePower_PCS_i = 0;  //第i个风光储单元的总升无功功率能力
    for (size_t j = 0; j < IntegrationUnitInput.PCS.size(); j = j + 1)
    {
        if (IntegrationUnitInput.PCS[j].Type_PCS == 1)
        {
            if (IntegrationUnitInput.PCS[j].RunStatus_PCS == 0)  //该PCS当前处于停机状态
            {
                Capacity_IncreaseReactivePower_PCS_i = 0;
            }
            else if (IntegrationUnitInput.PCS[j].RunStatus_PCS == 1)  //该PCS当前处于运行状态
            {
                if (IntegrationUnitInput.PCS[j].RunMode_PCS == 2)  //该PCS当前处于PQ运行模式
                {
                    double Qmax = sqrt(IntegrationUnitInput.PCS[j].Pn * IntegrationUnitInput.PCS[j].Pn - IntegrationUnitOutput.PCS[j].P_set * IntegrationUnitOutput.PCS[j].P_set);
                    Capacity_IncreaseReactivePower_PCS_i = Qmax - IntegrationUnitInput.PCS[j].Q_real;  //
                    if (Capacity_IncreaseReactivePower_PCS_i < 0)  //若超出PCS设备本身的最大升功率能力
                    {
                        Capacity_IncreaseReactivePower_PCS_i = 0;
                    }
                }
                else
                {
                    Capacity_IncreaseReactivePower_PCS_i = 0;  //单个风光储单元的总升功率能力=0
                }
            }
        }
        else
        {
            continue;
        }
    }
    return Capacity_IncreaseReactivePower_PCS_i;
}

//风光储单元的总减无功功率调节能力评估函数
double HaikouMicrogridEMSAlgorithm::PCSDecreaseReactivePowerCapacityCalculateFunction(Integration_Input IntegrationUnitInput, Integration_Output IntegrationUnitOutput)
{
    double Capacity_DecreaseReactivePower_PCS_i = 0;  //第i个风光储单元的总减无功功率能力
    for (size_t j = 0; j < IntegrationUnitInput.PCS.size(); j = j + 1)
    {
        if (IntegrationUnitInput.PCS[j].Type_PCS == 1)
        {
            if (IntegrationUnitInput.PCS[j].RunStatus_PCS == 0)  //该PCS当前处于停机状态
            {
                Capacity_DecreaseReactivePower_PCS_i = 0;
            }
            else if (IntegrationUnitInput.PCS[j].RunStatus_PCS == 1)  //该PCS当前处于运行状态
            {
                if (IntegrationUnitInput.PCS[j].RunMode_PCS == 2)  //该PCS当前处于PQ运行模式
                {
                    double Qmin = -sqrt(IntegrationUnitInput.PCS[j].Pn * IntegrationUnitInput.PCS[j].Pn - IntegrationUnitOutput.PCS[j].P_set * IntegrationUnitOutput.PCS[j].P_set);
                    Capacity_DecreaseReactivePower_PCS_i = Qmin - IntegrationUnitInput.PCS[j].Q_real;  //
                    if (Capacity_DecreaseReactivePower_PCS_i > 0)  //若超出PCS设备本身的最大减功率能力
                    {
                        Capacity_DecreaseReactivePower_PCS_i = 0;
                    }
                }
                else
                {
                    Capacity_DecreaseReactivePower_PCS_i = 0;  //单个风光储单元的总减功率能力=0
                }
            }
        }
        else
        {
            continue;
        }
    }
    return Capacity_DecreaseReactivePower_PCS_i;
}

//风光储一体化单元（单个PCS）无功功率设定值计算与修正函数
void HaikouMicrogridEMSAlgorithm::PCSReactivePowerSetCalculateFunction(PCS_Input PCSInput, PCS_Output& PCSOutput, double StepLimit)
{
    PCSOutput.Q_set=0;

    if (PCSInput.Type_PCS == 1)  //该PCS为用于连接负荷与风光储的双向变流器型PCS
    {
        if (fabs(PCSInput.Q_set_1 - PCSInput.Q_real) <= fabs(StepLimit))
        {
            PCSOutput.Q_set = PCSInput.Q_set_1 + PCSOutput.deltaQ_set;
        }
        else
        {
            PCSOutput.Q_set = PCSInput.Q_real + PCSOutput.deltaQ_set;
        }

        if (PCSOutput.Q_set > PCSInput.Pn)
        {
            PCSOutput.Q_set = PCSInput.Pn;
        }
        else if (PCSOutput.Q_set < -PCSInput.Pn)
        {
            PCSOutput.Q_set = -PCSInput.Pn;
        }
    }
}
//离网运行频率控制函数，待开发
void HaikouMicrogridEMSAlgorithm::OffgridOperationFrequencyControlFunction()
{

}

//离网运行电压控制函数，待开发
void HaikouMicrogridEMSAlgorithm::OffgridOperationVoltageControlFunction()
{

}
