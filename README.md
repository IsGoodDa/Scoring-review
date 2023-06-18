### 评分审核模块是发展性测评系统中的一个关键模块，提供了对用户提交的评价数据进行审核和管理的功能。管理员可以方便地对提交的评价数据进行审核和修改，并且可以根据需要进行统计和分析。这个模块可以有效地提高数据的准确性和安全性，保障测评结果的客观性和公正性。

## The rating review module is a key module in the developmental assessment system, providing the ability to review and manage user-submitted evaluation data. Administrators can easily review and modify submitted review data, and can perform statistics and analysis as needed. This module can effectively improve the accuracy and safety of data, and ensure the objectivity and fairness of assessment results.

### 以下是Java代码的一个示例；
## The following is an example of Java code：

package com.ruoyi.kx.controller;

import com.ruoyi.common.annotation.Log;

import com.ruoyi.common.core.controller.BaseController;

import com.ruoyi.common.core.domain.AjaxResult;

import com.ruoyi.common.core.domain.entity.SysRole;

import com.ruoyi.common.core.page.TableDataInfo;

import com.ruoyi.common.enums.BusinessType;

import com.ruoyi.common.utils.poi.ExcelUtil;

import com.ruoyi.kx.bean.KxScoreRecordBean;

import com.ruoyi.kx.bean.ScoreBean;

import com.ruoyi.kx.bean.ScoreSort;

import com.ruoyi.kx.bean.SelectStudentRecord;

import com.ruoyi.kx.domain.*;

import com.ruoyi.kx.mapper.*;

import com.ruoyi.kx.service.IKxScoreRecordService;

import org.springframework.beans.factory.annotation.Autowired;

import org.springframework.stereotype.Controller;

import org.springframework.ui.ModelMap;

import org.springframework.web.bind.annotation.*;

import org.springframework.web.multipart.MultipartFile;


import java.math.BigDecimal;

import java.math.RoundingMode;

import java.text.DecimalFormat;

import java.util.ArrayList;

import java.util.Date;

import java.util.List;

import java.util.stream.Collectors;


/**
 * 评价记录Controller
 *
 * @author huang
 * @date 2023-02-12
 */
@Controller
@RequestMapping("/kx/ScoreRecord")
public class KxScoreRecordController extends BaseController {
    private final String prefix = "kx/ScoreRecord";
    @Autowired
    KxScoreRecordMapper kxScoreRecordMapper;
    @Autowired
    KxStudentMapper kxStudentMapper;
    @Autowired
    KxTimingRateMapper kxTimingRateMapper;
    @Autowired
    KxRateRulesMapper kxRateRulesMapper;
    @Autowired
    KxScoreRecordMapper scoreRecordMapper;
    /**
     * 修改保存评价记录
     */

    @Autowired
    KxRateOftenMapper kxRateOftenMapper;
    @Autowired
    KxClassesMapper kxClassesMapper;
    @Autowired
    KxTimingMapper kxTimingMapper;
    @Autowired
    private IKxScoreRecordService kxScoreRecordService;

    @Autowired
    KxClassesAllotMapper kxClassesAllotMapper;


    /**
     * 查询评价记录列表
     */
    //@RequiresPermissions("kx:ScoreRecord:list")
    @PostMapping("/list")
    @ResponseBody
    public TableDataInfo list(KxScoreRecord kxScoreRecord) {
        startPage();
        List<KxScoreRecord> list = kxScoreRecordService.selectKxScoreRecordList(kxScoreRecord);
//        logger.info("list111:{}",list);
        return getDataTable(list);
    }

    /**
     * 查询学生评价记录
     */
    //@RequiresPermissions("kx:ScoreRecord:list")
    @PostMapping("/select")
    @ResponseBody
    public AjaxResult select(KxScoreRecord kxScoreRecord) {
        List<KxScoreRecord> list = kxScoreRecordService.selectKxScoreRecordList(kxScoreRecord);
        return success().put("detail", list.size()).put("rows", list);
    }

    /**
     * 查询学生评价记录
     */
    //@RequiresPermissions("kx:ScoreRecord:list")
    @GetMapping("/teacherSelect")
    @ResponseBody
    public AjaxResult select1(Long classId) {
        List<KxScoreRecord> list = kxScoreRecordMapper.selectKxScoreRecordByClassId(classId);

        //不查询老师的的评价记录
        list = list.stream().filter(kxScoreRecord1 -> !kxScoreRecord1.getVerify().equals("教师")).collect(Collectors.toList());
        return success().put("detail", list.size()).put("rows", list);
    }

    @Autowired
    KxTeacherMapper kxTeacherMapper;


    @GetMapping("/sort/{classId}/{type}/{id}")
    @ResponseBody
    public AjaxResult sort(@PathVariable Long classId, @PathVariable String type, @PathVariable Long id) {
        List<KxStudent> kxStudents = kxStudentMapper.selectKxStudentByClassId(classId);
//        KxTimingRate kxTimingRate = kxTimingRateMapper.selectKxTimingRateNow();
        KxTiming kxTiming = kxTimingMapper.selectKxTimingByNow();
        ArrayList<ScoreSort> scoreSorts = new ArrayList<>();
        for (KxStudent kxStudent : kxStudents) {
            ScoreSort scoreSort = kxScoreRecordMapper.selectKxScoreRecordBySort(kxStudent.getId(), type, id, kxTiming.getId());
            DecimalFormat df = new DecimalFormat("#.##"); // 创建 DecimalFormat 对象，指定格式为保留最多两位小数
            df.setRoundingMode(RoundingMode.DOWN); // 设置舍入模式为向下舍入
            float rounded;


            boolean NoType = !type.equals("A") && !type.equals("B") && !type.equals("D");
            if (type.equals("A") && id == 2 || type.equals("B") && id == 6) {
                float v = kxRateOftenMapper.selectKxScoreOftenAvRate(kxStudent.getId(), kxTiming.getId());
                rounded = Float.parseFloat(df.format(scoreSort.getScore() + v)); // 格式化小数部分并将结果转换为 float 类型

            } else if (NoType) {
                float v = kxRateOftenMapper.selectKxScoreOftenAvRate(kxStudent.getId(), kxTiming.getId());
                rounded = Float.parseFloat(df.format(scoreSort.getScore() + v)); // 格式化小数部分并将结果转换为 float 类型
            } else {
                rounded = Float.parseFloat(df.format(scoreSort.getScore())); // 格式化小数部分并将结果转换为 float 类型
            }

            if (type.equals("D") && id == 19) {
                float v = kxRateOftenMapper.selectKxScoreOftenAvRateByRateIdAndStudentId(kxStudent.getId(), kxTiming.getId(), 19L);
                rounded = Float.parseFloat(df.format(v));
            }

            if (type.equals("D") && id == 20) {
                float v = kxRateOftenMapper.selectKxScoreOftenAvRateByRateIdAndStudentId(kxStudent.getId(), kxTiming.getId(), 20L);
                rounded = Float.parseFloat(df.format(v));
            }

            if (type.equals("D") && id == 21) {
                float v = kxRateOftenMapper.selectKxScoreOftenAvRateByRateIdAndStudentId(kxStudent.getId(), kxTiming.getId(), 21L);
                rounded = Float.parseFloat(df.format(v));
            }

            String result = df.format(rounded);
            scoreSort.setStudentName(kxStudent.getName());
            scoreSort.setImagePath(kxStudent.getImagePath());
            scoreSort.setScore(Float.parseFloat(result));
            KxMark kxMark = kxStudent.getKxMark();

            if (kxMark != null) {
                char firstChar = kxStudent.getClasses().getName().charAt(0);
                if (firstChar == '一' || firstChar == '二') {

                    if (NoType) {
                        scoreSort.setScore(changeFloat(scoreSort.getScore() + getRateMarkScore(kxMark.getMoralScore(), true).floatValue() + getRateMarkScore(kxMark.getChineseScore(), true).floatValue() + getRateMarkScore(kxMark.getMathScore(), true).floatValue() + getRateMarkScore(kxMark.getScienceScore(), true).floatValue() + getRateMarkScore(kxMark.getPhysicalScore(), true).floatValue() + getRateMarkScore(kxMark.getArtScore(), true).floatValue() + getRateMarkScore(kxMark.getMusicScore(), true).floatValue() + getRateMarkScore(kxMark.getLaoDongScore(), true).floatValue()
//                                       getRateMarkScore(kxMark.getEnglishScore(), true).floatValue()
                        ));
                    }


                    if (type.equals("B") && id == 2) {
                        if (kxMark.getMoralScore() != null) {
                            scoreSort.setScore(getRateMarkScore(kxMark.getMoralScore(), true).floatValue());
                        }
                    }


                    if (type.equals("D") && id == 48) {
                        if (kxMark.getMoralScore() != null) {
                            scoreSort.setScore(getRateMarkScore(kxMark.getMoralScore(), true).floatValue());
                        }
                    }

                    if (type.equals("B") && id == 7) {
                        scoreSort.setScore(changeFloat(
//                            getRateMarkScore(kxMark.getMoralScore(), true).floatValue() +
                                getRateMarkScore(kxMark.getChineseScore(), true).floatValue() + getRateMarkScore(kxMark.getMathScore(), true).floatValue() + getRateMarkScore(kxMark.getScienceScore(), true).floatValue()
//                                    getRateMarkScore(kxMark.getPhysicalScore(), true).floatValue() +
//                                    getRateMarkScore(kxMark.getArtScore(), true).floatValue() +
//                                    getRateMarkScore(kxMark.getMusicScore(), true).floatValue() +
//                                    getRateMarkScore(kxMark.getLaoDongScore(), true).floatValue()
//                                    getRateMarkScore(kxMark.getEnglishScore(), true).floatValue()
                        ));
                    }
                    if (type.equals("B") && id == 9) {
                        scoreSort.setScore(changeFloat(
//                            getRateMarkScore(kxMark.getMoralScore(), true).floatValue() +
                                scoreSort.getScore() +
//                                     getRateMarkScore(kxMark.getChineseScore(), true).floatValue() +
//                                        getRateMarkScore(kxMark.getMathScore(), true).floatValue() +
//                                        getRateMarkScore(kxMark.getScienceScore(), true).floatValue()
                                        getRateMarkScore(kxMark.getPhysicalScore(), true).floatValue()
//                                    getRateMarkScore(kxMark.getArtScore(), true).floatValue() +
//                                    getRateMarkScore(kxMark.getMusicScore(), true).floatValue() +
//                                    getRateMarkScore(kxMark.getLaoDongScore(), true).floatValue()
//                                    getRateMarkScore(kxMark.getEnglishScore(), true).floatValue()
                        ));
                    }
                    if (type.equals("D") && id == 55) {
                        if (kxMark.getMoralScore() != null) {
                            scoreSort.setScore(getRateMarkScore(kxMark.getPhysicalScore(), true).floatValue());
                        }
                    }

                    if (type.equals("B") && id == 10) {
                        scoreSort.setScore(changeFloat(
//                            getRateMarkScore(kxMark.getMoralScore(), true).floatValue() +
//                                     getRateMarkScore(kxMark.getChineseScore(), true).floatValue() +
//                                        getRateMarkScore(kxMark.getMathScore(), true).floatValue() +
//                                        getRateMarkScore(kxMark.getScienceScore(), true).floatValue()
//                                        getRateMarkScore(kxMark.getPhysicalScore(), true).floatValue()
                                getRateMarkScore(kxMark.getArtScore(), true).floatValue() + getRateMarkScore(kxMark.getMusicScore(), true).floatValue()
//                                    getRateMarkScore(kxMark.getLaoDongScore(), true).floatValue()
//                                    getRateMarkScore(kxMark.getEnglishScore(), true).floatValue()
                        ));
                    }

                    if (type.equals("D") && id == 53) {
                        if (kxMark.getMoralScore() != null) {
                            scoreSort.setScore(getRateMarkScore(kxMark.getArtScore(), true).floatValue());
                        }
                    }
                    if (type.equals("D") && id == 54) {
                        if (kxMark.getMoralScore() != null) {
                            scoreSort.setScore(getRateMarkScore(kxMark.getMusicScore(), true).floatValue());
                        }
                    }

                } else {
                    logger.info("kxMark111:{}", kxMark.getMoralScore());

                    if (NoType) {
                        scoreSort.setScore(changeFloat(scoreSort.getScore() + getRateMarkScore(kxMark.getMoralScore(), false).floatValue() + getRateMarkScore(kxMark.getChineseScore(), false).floatValue() + getRateMarkScore(kxMark.getMathScore(), false).floatValue() + getRateMarkScore(kxMark.getScienceScore(), false).floatValue() + getRateMarkScore(kxMark.getPhysicalScore(), false).floatValue() + getRateMarkScore(kxMark.getArtScore(), false).floatValue() + getRateMarkScore(kxMark.getMusicScore(), false).floatValue() + getRateMarkScore(kxMark.getEnglishScore(), false).floatValue()));
                    }

                    if (type.equals("B") && id == 2) {
                        if (kxMark.getMoralScore() != null) {
                            scoreSort.setScore(getRateMarkScore(kxMark.getMoralScore(), false).floatValue());
                        }
                    }

                    if (type.equals("D") && id == 48) {
                        if (kxMark.getMoralScore() != null) {
                            scoreSort.setScore(getRateMarkScore(kxMark.getMoralScore(), false).floatValue());
                        }
                    }


                    if (type.equals("B") && id == 7) {
                        scoreSort.setScore(changeFloat(
//                                getRateMarkScore(kxMark.getMoralScore(), false).floatValue() +
                                getRateMarkScore(kxMark.getChineseScore(), false).floatValue() + getRateMarkScore(kxMark.getMathScore(), false).floatValue() + getRateMarkScore(kxMark.getEnglishScore(), false).floatValue() + getRateMarkScore(kxMark.getScienceScore(), false).floatValue()
//                                       getRateMarkScore(kxMark.getPhysicalScore(), false).floatValue() +
//                                       getRateMarkScore(kxMark.getArtScore(), false).floatValue() +
//                                          getRateMarkScore(kxMark.getMusicScore(), false).floatValue()
                        ));
                    }

                    if (type.equals("B") && id == 9) {
                        scoreSort.setScore(changeFloat(
//                            getRateMarkScore(kxMark.getMoralScore(), true).floatValue() +
                                scoreSort.getScore() +
//                                        getRateMarkScore(kxMark.getChineseScore(), false).floatValue() +
//                                        getRateMarkScore(kxMark.getMathScore(), false).floatValue() +
//                                        getRateMarkScore(kxMark.getScienceScore(), false).floatValue()
                                        getRateMarkScore(kxMark.getPhysicalScore(), false).floatValue()
//                                    getRateMarkScore(kxMark.getArtScore(), true).floatValue() +
//                                    getRateMarkScore(kxMark.getMusicScore(), true).floatValue() +
//                                    getRateMarkScore(kxMark.getLaoDongScore(), true).floatValue()
//                                    getRateMarkScore(kxMark.getEnglishScore(), true).floatValue()
                        ));
                    }

                    if (type.equals("D") && id == 55) {
                        if (kxMark.getMoralScore() != null) {
                            scoreSort.setScore(getRateMarkScore(kxMark.getPhysicalScore(), false).floatValue());
                        }
                    }

                    if (type.equals("B") && id == 10) {
                        scoreSort.setScore(changeFloat(
//                            getRateMarkScore(kxMark.getMoralScore(), true).floatValue() +
//                                     getRateMarkScore(kxMark.getChineseScore(), true).floatValue() +
//                                        getRateMarkScore(kxMark.getMathScore(), true).floatValue() +
//                                        getRateMarkScore(kxMark.getScienceScore(), true).floatValue()
//                                        getRateMarkScore(kxMark.getPhysicalScore(), true).floatValue()
                                getRateMarkScore(kxMark.getArtScore(), false).floatValue() + getRateMarkScore(kxMark.getMusicScore(), false).floatValue()
//                                    getRateMarkScore(kxMark.getLaoDongScore(), true).floatValue()
//                                    getRateMarkScore(kxMark.getEnglishScore(), true).floatValue()
                        ));
                    }

                    if (type.equals("D") && id == 53) {
                        if (kxMark.getMoralScore() != null) {
                            scoreSort.setScore(getRateMarkScore(kxMark.getArtScore(), false).floatValue());
                        }
                    }
                    if (type.equals("D") && id == 54) {
                        if (kxMark.getMoralScore() != null) {
                            scoreSort.setScore(getRateMarkScore(kxMark.getMusicScore(), false).floatValue());
                        }
                    }

                }

            }


            scoreSorts.add(scoreSort);
        }
        scoreSorts.sort(new ScoreSort());
        return success().put("detail", scoreSorts.size()).put("week", kxTiming).put("rows", scoreSorts);
    }


    public Float changeFloat(Float number) {
        DecimalFormat decimalFormat = new DecimalFormat("#.#");
        String formattedNumber = decimalFormat.format(number);
        return Float.parseFloat(formattedNumber);
    }

    public BigDecimal getRateMarkScore(BigDecimal score, boolean flag) {

        if (flag) {
            return score;
        }
        if (score != null) {
            if (score.compareTo(BigDecimal.valueOf(90)) >= 0) {
                return BigDecimal.valueOf(5);
            } else if (score.compareTo(BigDecimal.valueOf(80)) >= 0) {
                return BigDecimal.valueOf(4);
            } else if (score.compareTo(BigDecimal.valueOf(70)) >= 0) {
                return BigDecimal.valueOf(3);
            } else if (score.compareTo(BigDecimal.valueOf(60)) >= 0) {
                return BigDecimal.valueOf(2);
            } else if (score.compareTo(BigDecimal.valueOf(0)) > 0) {
                return BigDecimal.valueOf(1);
            } else {
                //英语分0分就是没考 可能是三年级以下
                return BigDecimal.valueOf(0);
            }
        } else {
            return BigDecimal.valueOf(0);
        }
    }


    /**
     * 查询学生评价记录
     */
    //@RequiresPermissions("kx:ScoreRecord:list")
    @GetMapping("/selectNew/{classId}/{status}/{startTime}/{endTime}")
    @ResponseBody
    public AjaxResult select3(@PathVariable Long classId, @PathVariable String status, @PathVariable String startTime, @PathVariable String endTime) {

        List<KxScoreRecord> list = kxScoreRecordMapper.selectKxScoreRecordByNewTwenty(classId, status, startTime, endTime);
        if (list.size() >= 20) {
            list = list.subList(0, 20);
        }
        return success().put("detail", list.size()).put("rows", list);
    }

    /**
     * 查询学生评价记录
     */
    //@RequiresPermissions("kx:ScoreRecord:list")
    @PostMapping("/selectFinish")
    @ResponseBody
    public AjaxResult select2(SelectStudentRecord param) {
        List<KxScoreRecord> kxScoreRecords = kxScoreRecordMapper.selectStudentRecord(param);
        if (kxScoreRecords != null && kxScoreRecords.size() > 0) {
            return success().put("detail", kxScoreRecords.size()).put("rows", kxScoreRecords);

        } else {
            return error("未找到对应记录");
        }
    }


    /**
     * 查询未完成周期评记录的学生 家长专用
     */
    //@RequiresPermissions("kx:ScoreRecord:list")
    @GetMapping("/select2/{status}/{studentId}/{KxTimingRateId}")
    @ResponseBody
    public AjaxResult select2(@PathVariable String status, @PathVariable Long studentId, @PathVariable Long KxTimingRateId) {
        //查询学生
        KxStudent kxStudent = kxStudentMapper.selectKxStudentById(studentId);
        KxScoreRecord kxScoreRecord;
        kxScoreRecord = new KxScoreRecord();
        kxScoreRecord.setStudentId(kxStudent.getId());
        kxScoreRecord.setVerify(status);
        kxScoreRecord.setClassId(kxStudent.getClassId());
        kxScoreRecord.setTimingRateId(KxTimingRateId);
        List<KxScoreRecord> kxScoreRecords = kxScoreRecordMapper.selectKxScoreRecordList(kxScoreRecord);
        KxTimingRate kxTimingRate = kxTimingRateMapper.selectKxTimingRateById(KxTimingRateId);
        if (kxScoreRecords.size() == 0) {
            return success().put("data", kxTimingRate);
        } else {
            return error();
        }
    }

    /**
     * 导出评价记录列表
     */
    //@RequiresPermissions("kx:ScoreRecord:export")
    @Log(title = "评价记录", businessType = BusinessType.EXPORT)
    @PostMapping("/export")
    @ResponseBody
    public AjaxResult export(KxScoreRecord kxScoreRecord) {
        List<KxScoreRecord> list = kxScoreRecordService.selectKxScoreRecordList(kxScoreRecord);
        ExcelUtil<KxScoreRecord> util = new ExcelUtil<KxScoreRecord>(KxScoreRecord.class);
        return util.exportExcel(list, "评价记录数据");
    }

    @Log(title = "评价记录", businessType = BusinessType.IMPORT)
    //@RequiresPermissions("kx:ScoreRecord:import")
    @PostMapping("/importData")
    @ResponseBody
    public AjaxResult importData(MultipartFile file, boolean updateSupport) throws Exception {
        ExcelUtil<KxScoreRecord> util = new ExcelUtil<KxScoreRecord>(KxScoreRecord.class);
        List<KxScoreRecord> kxScoreRecordList = util.importExcel(file.getInputStream());
        String message = kxScoreRecordService.importKxScoreRecord(kxScoreRecordList, updateSupport, getLoginName());
        return AjaxResult.success(message);
    }

    //@RequiresPermissions("kx:ScoreRecord:view")
    @GetMapping("/importTemplate")
    @ResponseBody
    public AjaxResult importTemplate() {
        ExcelUtil<KxScoreRecord> util = new ExcelUtil<KxScoreRecord>(KxScoreRecord.class);
        return util.importTemplateExcel(" 评价记录数据");
    }

    //@RequiresPermissions("kx:ScoreRecord:view")
    @GetMapping()
    public String ScoreRecord(ModelMap modelMap) {
        modelMap.put("loginName", getSysUser().getLoginName());
        List<KxClasses> kxClasses = kxClassesMapper.selectKxClassesList(null);
        kxClasses = kxClasses.subList(1, kxClasses.size());
        logger.info("kxClass111:{}", kxClasses);

        boolean flag = false;
        for (SysRole role : getSysUser().getRoles()) {
            Long roleId = role.getRoleId();
            if (roleId == 5) {
                flag = true;
                break;
            }
        }
        modelMap.put("flag", flag);
        if (!flag && !getSysUser().getLoginName().equals("admin")) {

            kxClasses = new ArrayList<>();
            for (SysRole role : getSysUser().getRoles()) {
                if (role.getRoleId() == 6) {
                    List<KxClassesAllot> kxClassesAllots = kxClassesAllotMapper.selectKxClassesAllotByTeacherId(kxTeacherMapper.selectKxTeacherByUserId(getUserId()).getId());
                    for (KxClassesAllot kxClassesAllot : kxClassesAllots) {
                        kxClasses.add(kxClassesMapper.selectKxClassesById(kxClassesAllot.getClassId()));
                    }
                } else if (role.getRoleId() == 2) {
                    KxTeacher kxTeacher = kxTeacherMapper.selectKxTeacherByUserId(getUserId());
                    kxClasses.add(kxClassesMapper.selectKxClassesByTeacherId(kxTeacher.getId()));
                }
            }

            logger.info("kxClass111:{}", kxClasses);
        }
        modelMap.put("classes", kxClasses);
        modelMap.put("timing", kxTimingMapper.selectKxTimingList(null));
        modelMap.put("rate", kxRateRulesMapper.selectKxRateRulesList(null));
        return prefix + "/ScoreRecord";
    }

    /**
     * 新增评价记录
     */
    @GetMapping("/add")
    public String add(ModelMap modelMap) {
        modelMap.put("loginName", getSysUser().getLoginName());
        List<KxClasses> kxClasses = kxClassesMapper.selectKxClassesList(null);
        kxClasses = kxClasses.subList(1, kxClasses.size());
        logger.info("kxClass111:{}", kxClasses);

        boolean flag = false;
        for (SysRole role : getSysUser().getRoles()) {
            Long roleId = role.getRoleId();
            if (roleId == 5) {
                flag = true;
                break;
            }
        }
        modelMap.put("flag", flag);

        if (!flag && !getSysUser().getLoginName().equals("admin")) {

            kxClasses = new ArrayList<>();
            for (SysRole role : getSysUser().getRoles()) {
                if (role.getRoleId() == 6) {
                    List<KxClassesAllot> kxClassesAllots = kxClassesAllotMapper.selectKxClassesAllotByTeacherId(kxTeacherMapper.selectKxTeacherByUserId(getUserId()).getId());
                    for (KxClassesAllot kxClassesAllot : kxClassesAllots) {
                        kxClasses.add(kxClassesMapper.selectKxClassesById(kxClassesAllot.getClassId()));
                    }
                } else if (role.getRoleId() == 2) {
                    KxTeacher kxTeacher = kxTeacherMapper.selectKxTeacherByUserId(getUserId());
                    kxClasses.add(kxClassesMapper.selectKxClassesByTeacherId(kxTeacher.getId()));
                }
            }

            logger.info("kxClass111:{}", kxClasses);
        }
        modelMap.put("classes", kxClasses);
        modelMap.put("timing", kxTimingMapper.selectKxTimingList(null));
        modelMap.put("rate", kxRateRulesMapper.selectKxRateRulesList(null));
        return prefix + "/add";
    }

    /**
     * 新增保存评价记录
     */
    //@RequiresPermissions("kx:ScoreRecord:add")
    @Log(title = "评价记录", businessType = BusinessType.INSERT)
    @PostMapping("/add")
    @ResponseBody
    public AjaxResult addSave(KxScoreRecord kxScoreRecord) {
        return toAjax(kxScoreRecordService.insertKxScoreRecord(kxScoreRecord));
    }


    /**
     * 查询未完成周期评记录的学生列表
     */
    //@RequiresPermissions("kx:ScoreRecord:list")
    @GetMapping("/select/{rateId}/{classId}")
    @ResponseBody
    public AjaxResult select(@PathVariable Long rateId, @PathVariable Long classId) {

        //查询班级所有学生
        List<KxStudent> kxStudents = kxStudentMapper.selectKxStudentByClassId(classId);
//        KxTimingRate kxTimingRate = kxTimingRateMapper.selectKxTimingRateNow();


        KxTiming kxTiming = kxTimingMapper.selectKxTimingByNow();

        KxRateRules kxRateRule = kxRateRulesMapper.selectKxRateRulesById(rateId);
        String type = kxRateRule.getPeriod();
        //获取本周评信息
        KxTimingRate kxTimingRate = kxTimingRateMapper.selectKxTimingRateNow2(kxTiming.getId(), type);

        if (kxTimingRate == null) {
            return error("未到" + type + "时间，无法评价");
        }

        KxClasses kxClasses = kxClassesMapper.selectKxClassesById(classId);

        char firstChar = kxClasses.getName().charAt(0);
        logger.info("firstChar111:{}", firstChar);
        if (firstChar != '一' && firstChar != '二') {
            logger.info("firstChar222:{}", firstChar);
            List<KxRateRules> kxRateRules = kxRateRulesMapper.selectKxRateRulesByParentId(15L);
            for (KxRateRules i : kxRateRules) {
                if (i.getId().equals(rateId)) {
                    return error("该评价只有一二年级可以评价");

                }

            }

        }
        KxScoreRecord kxScoreRecord;
        for (KxStudent kxStudent : kxStudents) {
            kxScoreRecord = new KxScoreRecord();
            kxScoreRecord.setStudentId(kxStudent.getId());
            kxScoreRecord.setRateId(rateId);
            kxScoreRecord.setClassId(classId);
            kxScoreRecord.setTimingRateId(kxTimingRate.getId());
            float lastScore = kxScoreRecordMapper.selectKxScoreRecordLastRate(kxStudent.getId(), rateId);
            kxStudent.setLastScore(lastScore);
            float avScore = kxScoreRecordMapper.selectKxScoreRecordAvRate(kxStudent.getId(), rateId);
            float totalScore = kxScoreRecordMapper.selectKxScoreRecordTotalRate(kxStudent.getId(), rateId);
            kxStudent.setAvScore(avScore);
            kxStudent.setTotalScore(totalScore);
            kxStudent.setClasses(null);
            kxStudent.setKxMark(null);
        }
        return success().put("detail", kxStudents.size()).put("rows", kxStudents).put("week", kxTimingRate);
    }

    /**
     * 提交保存评价记录
     */
    //@RequiresPermissions("kx:ScoreRecord:add")
    @Log(title = "评价记录", businessType = BusinessType.INSERT)
    @PostMapping("/submit")
    @ResponseBody
    public AjaxResult submit(@RequestBody KxSubmitRate kxSubmitRate) {
//        KxTimingRate kxTimingRate = kxTimingRateMapper.selectKxTimingRateNow();

        KxTiming kxTiming = kxTimingMapper.selectKxTimingByNow();
        Long rateId = kxSubmitRate.getRateId();
        KxRateRules kxRateRule = kxRateRulesMapper.selectKxRateRulesById(rateId);
        String type = kxRateRule.getPeriod();
        //获取本周评信息
        KxTimingRate kxTimingRate = kxTimingRateMapper.selectKxTimingRateNow2(kxTiming.getId(), type);

        KxStudent tempKxStudent = kxStudentMapper.selectKxStudentById(kxSubmitRate.getKxStudents().get(0).getId());

        if (kxTimingRate == null) {
            return error("未到" + type + "时间，无法评价");
        }


        List<KxScoreRecord> kxScoreRecords = kxScoreRecordMapper.selectKxScoreRecordByClassIdandRateIdandTimingRateId(tempKxStudent.getClassId(), rateId, kxTimingRate.getId());


        if (kxScoreRecords.size() == 0) {
            logger.info("kxScoreRecords6666:{}", kxScoreRecords);

        } else {
            logger.info("kxScoreRecords7777:{}", kxScoreRecords);
        }

     /*   if (kxScoreRecords.size() != 0) {

            return error("本周已经提交过评价啦");
        } else {
            logger.info("kxScoreRecords222:{}", kxScoreRecords);

        }*/

        KxStudent statusStudent = kxStudentMapper.selectKxStudentByUserId(getUserId());
        if (statusStudent != null) {
            for (KxStudent kxStudent : kxSubmitRate.getKxStudents()) {
                KxScoreRecord kxScoreRecord = new KxScoreRecord();
                kxScoreRecord.setStudentId(kxStudent.getId());
                kxScoreRecord.setValuatorId(statusStudent.getId());
                kxScoreRecord.setTimingRateId(kxTimingRate.getId());
                kxScoreRecord.setVerify(statusStudent.getStatus());
                kxScoreRecord.setClassId(statusStudent.getClassId());
                kxScoreRecord.setRateId(rateId);
                kxScoreRecord.setRateScore(kxStudent.getScore());
                kxScoreRecordService.insertKxScoreRecord(kxScoreRecord);
            }
            return success("提交成功，等待老师审核");
        } else {
            logger.info("getSysUser:{}", getSysUser().toString());
            KxTeacher kxTeacher = kxTeacherMapper.selectKxTeacherByUserId(getUserId());
            if (kxTeacher == null) {
                return error("未找到对应系统用户:" + getSysUser().getLoginName());
            }
            KxStudent tempStudent = kxSubmitRate.getKxStudents().get(0);
            Long classId = null;
            if (tempStudent != null) {
                classId = kxStudentMapper.selectKxStudentById(tempStudent.getId()).getClassId();
            }
            for (KxStudent kxStudent : kxSubmitRate.getKxStudents()) {
                KxScoreRecord kxScoreRecord = new KxScoreRecord();
                kxScoreRecord.setStudentId(kxStudent.getId());
                kxScoreRecord.setValuatorId(kxTeacher.getId());
                kxScoreRecord.setTimingRateId(kxTimingRate.getId());
                kxScoreRecord.setVerify("教师");
                kxScoreRecord.setClassId(classId);
                kxScoreRecord.setRateId(rateId);
                kxScoreRecord.setRateScore(kxStudent.getScore());
                kxScoreRecord.setStatus(1L);
                kxScoreRecordService.insertKxScoreRecord(kxScoreRecord);
            }
            return success();
        }
    }

    @PostMapping("/editScore")
    @ResponseBody
    public AjaxResult editScore(KxSubmitRate kxSubmitRate) {
        Long deptId = getSysUser().getDeptId();

        KxScoreRecord kxScoreRecord = new KxScoreRecord();
        kxScoreRecord.setId(kxSubmitRate.getId());
        kxScoreRecord.setRateScore(kxSubmitRate.getScore());
        if (deptId == 101) {
            kxScoreRecord.setStatus(1L);
            kxScoreRecordService.updateKxScoreRecord(kxScoreRecord);
            return success("修改成功");
        } else {
            kxScoreRecord.setStatus(0L);
            kxScoreRecordService.updateKxScoreRecord(kxScoreRecord);
            return success("修改成功，等待老师审核");
        }
    }


    /**
     * 提交保存评价记录 给老师家长用的、 别删了！
     */
    //@RequiresPermissions("kx:ScoreRecord:add")
    @Log(title = "评价记录", businessType = BusinessType.INSERT)
    @PostMapping("/submit2")
    @ResponseBody
    public AjaxResult submit2(@RequestBody KxScoreRecordBean kxScoreRecordBean) {
        logger.info("kxScoreRecordBean:{}", kxScoreRecordBean);
        KxScoreRecord kxScoreRecord = new KxScoreRecord();
//      KxTimingRate kxTimingRate = kxTimingRateMapper.selectKxTimingRateNow();
        kxScoreRecord.setStudentId(kxScoreRecordBean.getBeStudentId());
        kxScoreRecord.setClassId(kxScoreRecordBean.getClassId());
        kxScoreRecord.setTimingRateId(kxScoreRecordBean.getTimingRateId());
        kxScoreRecord.setVerify(kxScoreRecordBean.getStatus());
        ArrayList<ScoreBean> scoreList = kxScoreRecordBean.getScoreList();
        boolean flag = false;
        for (ScoreBean scoreBean : scoreList) {
            kxScoreRecord.setId(null);
            kxScoreRecord.setRateId(scoreBean.getRateId());
            List<KxScoreRecord> kxScoreRecords = kxScoreRecordMapper.selectKxScoreRecordList2(kxScoreRecord);
            kxScoreRecord.setRateScore(scoreBean.getScore());
            logger.info("kxScoreRecordBean333:{}", kxScoreRecords.size());
            kxScoreRecord.setStatus(1L);
            if (kxScoreRecords.size() != 0) {
                scoreRecordMapper.updateKxScoreRecord2(kxScoreRecord);
                flag = true;
            } else {
                kxScoreRecordService.insertKxScoreRecord(kxScoreRecord);
            }
        }
        if (flag) {
            return success("已重新提交");
        } else {
            return success("提交成功");
        }
    }

    @Log(title = "评价记录", businessType = BusinessType.INSERT)
    @PostMapping("/parentSubmit")
    @ResponseBody
    public AjaxResult parentSubmit(@RequestBody KxScoreRecord kxScoreRecord) {


        KxStudent kxStudent = kxStudentMapper.selectKxStudentById(kxScoreRecord.getStudentId());
        KxRateRules kxRateRule = kxRateRulesMapper.selectKxRateRulesById(kxScoreRecord.getRateId());
        String type = kxRateRule.getPeriod();

        //获取本周评信息
        KxTiming kxTiming = kxTimingMapper.selectKxTimingByNow();
        KxTimingRate kxTimingRate = kxTimingRateMapper.selectKxTimingRateNow2(kxTiming.getId(), type);

        if (kxTimingRate == null) {
            return error("未到" + type + "时间，无法评价");
        }

        KxScoreRecord temp = new KxScoreRecord();
        temp.setStudentId(kxStudent.getId());
        temp.setClassId(kxStudent.getClassId());
        temp.setTimingRateId(kxTimingRate.getId());
        temp.setVerify("家长");
        temp.setRateScore(kxScoreRecord.getRateScore());
        temp.setRateId(kxScoreRecord.getRateId());
        temp.setStatus(0L);
        temp.setCreateTime(new Date());


        KxScoreRecord scoreRecord = kxScoreRecordMapper.selectKxScoreRecordByStudentIdAndRateIdAndTimingRateId(kxStudent.getId(), kxScoreRecord.getRateId(), kxTimingRate.getId());

        if (scoreRecord != null) {
            kxScoreRecordMapper.deleteKxImagesByParentId(scoreRecord.getId());
        }

        kxScoreRecordMapper.insertKxScoreRecord(temp);
        logger.info("temp111:{}", temp);
        if (kxScoreRecord.getImagesList() != null && kxScoreRecord.getImagesList().size() != 0) {
            for (kxImages kxImages : kxScoreRecord.getImagesList()) {
                kxImages.setParentId(temp.getId());
                kxScoreRecordMapper.insertKxImages(kxImages);

            }
        }

        if (scoreRecord != null) {
            return success("更新成功，等待老师审核");
        } else {
            return success("提交成功，等待老师审核");
        }
    }


    /**
     * 修改评价记录
     */
    //@RequiresPermissions("kx:ScoreRecord:edit")
    @GetMapping("/edit/{id}")
    public String edit(@PathVariable("id") Long id, ModelMap mmap) {
        KxScoreRecord kxScoreRecord = kxScoreRecordService.selectKxScoreRecordById(id);
        mmap.put("kxScoreRecord", kxScoreRecord);
        mmap.put("classes", kxClassesMapper.selectKxClassesList(null));
        mmap.put("timing", kxTimingMapper.selectKxTimingList(null));
        mmap.put("rate", kxRateRulesMapper.selectKxRateRulesList(null));
        return prefix + "/edit";
    }

    @GetMapping("/selectStudent/{classId}")
    @ResponseBody
    public AjaxResult selectStudent(@PathVariable Long classId) {

        List<KxStudent> kxStudents = kxStudentMapper.selectKxStudentByClassId(classId);
        return success().put("rows", kxStudents);
    }

    //@RequiresPermissions("kx:ScoreRecord:edit")
    @Log(title = "评价记录", businessType = BusinessType.UPDATE)
    @PostMapping("/edit")
    @ResponseBody
    public AjaxResult editSave(KxScoreRecord kxScoreRecord) {
        kxScoreRecordService.updateKxScoreRecord(kxScoreRecord);
        return success();
    }

    @Log(title = "评价记录", businessType = BusinessType.UPDATE)
    @PostMapping("/allSubmit")
    @ResponseBody
    public AjaxResult allSubmit(Long status, Long classId) {
        return success(kxScoreRecordMapper.updateAllKxScoreRecord(status, classId));
    }

    /**
     * 删除评价记录
     */
    //@RequiresPermissions("kx:ScoreRecord:remove")
    @Log(title = "评价记录", businessType = BusinessType.DELETE)
    @PostMapping("/remove")
    @ResponseBody
    public AjaxResult remove(String ids) {
        return toAjax(kxScoreRecordService.deleteKxScoreRecordByIds(ids));
    }
}


# <body>
  <header>
    <div class="logo">My Github Page</div>
    <nav>
      <a href="#">Home</a>
      <a href="#">About</a>
      <a href="#">Contact</a>
    </nav>
  </header>
  <h1>Welcome to My Github Page</h1>
  <footer>&copy; 2023 My Github Page. All rights reserved.</footer>
</body>
</html>
