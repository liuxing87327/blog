title: "使用pinyin4j将中文转换为拼音"
date: 2012-08-16 00:20
category: Java
tags: pinyin4j

---
直接代码

##实现一

```java
import net.sourceforge.pinyin4j.PinyinHelper;
import net.sourceforge.pinyin4j.format.HanyuPinyinCaseType;
import net.sourceforge.pinyin4j.format.HanyuPinyinOutputFormat;
import net.sourceforge.pinyin4j.format.HanyuPinyinToneType;
import net.sourceforge.pinyin4j.format.HanyuPinyinVCharType;
import net.sourceforge.pinyin4j.format.exception.BadHanyuPinyinOutputFormatCombination;

public class SpellHelper {

	/**
	 * 将中文转换为拼音
	 * @param name
	 * @return
	 */
	public static String getEname(String name) {
		HanyuPinyinOutputFormat pyFormat = new HanyuPinyinOutputFormat();
		pyFormat.setCaseType(HanyuPinyinCaseType.LOWERCASE);
		pyFormat.setToneType(HanyuPinyinToneType.WITHOUT_TONE);
		pyFormat.setVCharType(HanyuPinyinVCharType.WITH_V);

		try {
			return PinyinHelper.toHanyuPinyinString(name, pyFormat, "");
		} catch (BadHanyuPinyinOutputFormatCombination e) {
			e.printStackTrace();
		}
		return "";
	}
	
	/**
	 * 取字符拼音的第一个字母
	 * @param name
	 * @return
	 */
	public static String getFirstEName(String name) {
		if(DyString.isEmpty(name))
			return "";
		
		String spell = "";
		String _name = name.replaceAll("[　― …《》\\.,，。;`；\\!！\\?？‘’＜＞\"“”、■#★●【】\\{\\}\\[\\]（）+｛｝［］\\(\\)\\|\\-:：~～!@#$%^&*\\(\\)/\\,\\.￥——]", "").trim();
		
		char[] strs = _name.toCharArray();
		for (int i = 0; i < strs.length; i++) {
			String upCaseText = getEname("" + strs[i]);
			spell += upCaseText.substring(0, 1);
		}
		return spell;
	}
	
	public static void main(String[] args) {
		System.out.println(getEname("长沙阁酒店公寓（瑞苑）"));
		System.out.println(getFirstEName("长沙阁酒店公寓（瑞苑）"));
	}

}
```

##实现二

```java
import java.util.ArrayList;
import java.util.Hashtable;
import java.util.List;
import java.util.Map;

import net.sourceforge.pinyin4j.PinyinHelper;
import net.sourceforge.pinyin4j.format.HanyuPinyinCaseType;
import net.sourceforge.pinyin4j.format.HanyuPinyinOutputFormat;
import net.sourceforge.pinyin4j.format.HanyuPinyinToneType;
import net.sourceforge.pinyin4j.format.HanyuPinyinVCharType;
import net.sourceforge.pinyin4j.format.exception.BadHanyuPinyinOutputFormatCombination;

/**
 * 汉字转换拼音工具类
 * 
 * @Author liuxing
 * @Date 2013-03-14
 */
public class PinYin4JCn {
	
	private static HanyuPinyinOutputFormat spellFormat = null;
	private static String reg = "[　― …《》\\.,，。;`；\\!！\\?？‘’＜＞\"“”、■#★●【】\\{\\}\\[\\]（）+｛｝［］\\(\\)\\|\\-:：~～!@#$%^&*\\(\\)/\\,\\.￥——]";
	
	/**
	 * 将中文转换为拼音
	 * 不区分多音字
	 * @param name
	 * @return
	 */
	@SuppressWarnings("deprecation")
	public static String converterNoSpaceSpell(String name) {
		if(spellFormat == null){
			spellFormat = new HanyuPinyinOutputFormat();
			spellFormat.setCaseType(HanyuPinyinCaseType.LOWERCASE);
			spellFormat.setToneType(HanyuPinyinToneType.WITHOUT_TONE);
			spellFormat.setVCharType(HanyuPinyinVCharType.WITH_V);
		}

		try {
			return PinyinHelper.toHanyuPinyinString(name, spellFormat, "");
		} catch (BadHanyuPinyinOutputFormatCombination e) {
			e.printStackTrace();
		}
		return "";
	}
	
	/**
	 * 将中文转换为全拼（带空格）
	 * 不区分多音字
	 * @param name
	 * @return
	 */
	@SuppressWarnings("deprecation")
	public static String converterSpell(String name) {
		if(spellFormat == null){
			spellFormat = new HanyuPinyinOutputFormat();
			spellFormat.setCaseType(HanyuPinyinCaseType.LOWERCASE);
			spellFormat.setToneType(HanyuPinyinToneType.WITHOUT_TONE);
			spellFormat.setVCharType(HanyuPinyinVCharType.WITH_V);
		}
		
		try {
			return PinyinHelper.toHanyuPinyinString(name, spellFormat, " ");
		} catch (BadHanyuPinyinOutputFormatCombination e) {
			e.printStackTrace();
		}
		return "";
	}
	
	/**
	 * 取字符拼音的第一个字母
	 * 不区分多音字
	 * @param name
	 * @return
	 */
	public static String converterToFirstSpell(String name) {
		if(Strings.isEmpty(name)){
			return "";
		}
		
		String spell = "";
		String _name = name.replaceAll(reg, "").trim();
		
		char[] strs = _name.toCharArray();
		String upCaseText = null;
		for (int i = 0; i < strs.length; i++) {
			upCaseText = converterNoSpaceSpell("" + strs[i]);
			spell += upCaseText.substring(0, 1);
		}
		return spell;
	}
	
	/**
	 * 汉字转换位汉语拼音首字母，英文字符不变，
	 * 特殊字符丢失 支持多音字，生成方式如（重当参:cdc,zds,cds,zdc）
	 * @param chines
	 * @return
	 */
	public static String converterToFirstSpellWithPolyphone(String chines) {
		StringBuffer pinyinName = new StringBuffer();
		char[] nameChar = chines.toCharArray();
		HanyuPinyinOutputFormat defaultFormat = new HanyuPinyinOutputFormat();
		defaultFormat.setCaseType(HanyuPinyinCaseType.LOWERCASE);
		defaultFormat.setToneType(HanyuPinyinToneType.WITHOUT_TONE);
		for (int i = 0; i < nameChar.length; i++) {
			if (nameChar[i] > 128) {
				try {
					// 取得当前汉字的所有全拼
					String[] strs = PinyinHelper.toHanyuPinyinStringArray(nameChar[i], defaultFormat);
					if (strs != null) {
						for (int j = 0; j < strs.length; j++) {
							// 取首字母
							pinyinName.append(strs[j].charAt(0));
							if (j != strs.length - 1) {
								pinyinName.append(",");
							}
						}
					}
//					else {
//						pinyinName.append(nameChar[i]);
//					}
				} catch (BadHanyuPinyinOutputFormatCombination e) {
					e.printStackTrace();
				}
			} else {
				pinyinName.append(nameChar[i]);
			}
			pinyinName.append(" ");
		}
		return parseTheChineseByObject(discountTheChinese(pinyinName.toString()));
	}

	/**
	 * 汉字转换位汉语全拼，英文字符不变，特殊字符丢失
	 * 支持多音字，生成方式如（重当参:zhongdangcen,zhongdangcan,chongdangcen,chongdangshen,zhongdangshen,chongdangcan）
	 * @param chines
	 * @return
	 */
	public static String converterToSpellWithPolyphone(String chines) {
		StringBuffer pinyinName = new StringBuffer();
		char[] nameChar = chines.toCharArray();
		HanyuPinyinOutputFormat defaultFormat = new HanyuPinyinOutputFormat();
		defaultFormat.setCaseType(HanyuPinyinCaseType.LOWERCASE);
		defaultFormat.setToneType(HanyuPinyinToneType.WITHOUT_TONE);
		for (int i = 0; i < nameChar.length; i++) {
			if (nameChar[i] > 128) {
				try {
					// 取得当前汉字的所有全拼
					String[] strs = PinyinHelper.toHanyuPinyinStringArray(
							nameChar[i], defaultFormat);
					if (strs != null) {
						for (int j = 0; j < strs.length; j++) {
							pinyinName.append(strs[j]);
							if (j != strs.length - 1) {
								pinyinName.append(",");
							}
						}
					}
				} catch (BadHanyuPinyinOutputFormatCombination e) {
					e.printStackTrace();
				}
			} else {
				pinyinName.append(nameChar[i]);
			}
			pinyinName.append(" ");
		}
		return parseTheChineseByObject(discountTheChinese(pinyinName.toString()));
	}

	/**
	 * 
	 * 功能说明：获取所有的优化结果集
	 * 刘兴  2013-8-7 下午5:19:24
	 * 修改者名字：
	 * 修改日期：
	 * 修改内容：
	 * @param str
	 * @return
	 */
	public static String converterToAllSpellWithPolyphone(String str){
		String firstSpell = converterToFirstSpellWithPolyphone(str);
		String spell = converterToSpellWithPolyphone(str);
		
		if(Strings.isNotEmpty(firstSpell)){
			firstSpell = firstSpell + ",";
		}
		return firstSpell + spell;
	}
	
	/**
	 * 去除多音字重复数据
	 * @param theStr
	 * @return
	 */
	private static List<Map<String, Integer>> discountTheChinese(String theStr) {
		// 去除重复拼音后的拼音列表
		List<Map<String, Integer>> mapList = new ArrayList<Map<String, Integer>>();
		// 用于处理每个字的多音字，去掉重复
		Map<String, Integer> onlyOne = null;
		String[] firsts = theStr.split(" ");
		// 读出每个汉字的拼音
		for (String str : firsts) {
			onlyOne = new Hashtable<String, Integer>();
			String[] china = str.split(",");
			// 多音字处理
			for (String s : china) {
				Integer count = onlyOne.get(s);
				if (count == null) {
					onlyOne.put(s, new Integer(1));
				} else {
					onlyOne.remove(s);
					count++;
					onlyOne.put(s, count);
				}
			}
			mapList.add(onlyOne);
		}
		return mapList;
	}

	/**
	 * 解析并组合拼音，对象合并方案
	 * @param list
	 * @return
	 */
	private static String parseTheChineseByObject(List<Map<String, Integer>> list) {
		Map<String, Integer> first = null; // 用于统计每一次,集合组合数据

		for (int i = 0; i < list.size(); i++) {
			// 每一组集合与上一次组合的Map
			Map<String, Integer> temp = new Hashtable<String, Integer>();
			
			// 第一次循环，first为空
			if (first != null) {
				// 取出上次组合与此次集合的字符，并保存
				for (String s : first.keySet()) {
					for (String s1 : list.get(i).keySet()) {
						temp.put(s + s1, 1);
					}
				}
				
				// 清理上一次组合数据
				if (temp != null && temp.size() > 0) {
					first.clear();
				}
			} else {
				for (String s : list.get(i).keySet()) {
					temp.put(s, 1);
				}
			}
			
			// 保存组合数据以便下次循环使用
			if (temp != null && temp.size() > 0) {
				first = temp;
			}
		}
		
		if (first == null) 
			return "";
		
		String reltStr = "";
		
		for (String str : first.keySet()) {
			reltStr += ("," + str);
		}
		
		if (Strings.isNotEmpty(reltStr)) {
			return reltStr.replaceFirst(",", "");
		}
		
		return reltStr;
	}

	public static void main(String[] args) {
		String str = "单田芳";
//		System.out.println(converterToFirstSpellWithPolyphone(str));
//		System.out.println(converterToSpellWithPolyphone(str));
//		System.out.println(converterSpell(str));
		System.out.println(converterToFirstSpellWithPolyphone(str));
	}

}
```