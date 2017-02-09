package com.hy.weixin.utils;

import java.io.BufferedInputStream;
import java.io.ByteArrayInputStream;
import java.io.IOException;
import java.io.InputStream;
import java.io.OutputStreamWriter;
import java.io.UnsupportedEncodingException;
import java.net.HttpURLConnection;
import java.net.URL;
import java.net.URLEncoder;
import java.security.MessageDigest;
import java.security.NoSuchAlgorithmException;
import java.util.Formatter;
import java.util.HashMap;
import java.util.Map;
import java.util.Random;

import javax.xml.parsers.ParserConfigurationException;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.xml.sax.SAXException;

import com.alibaba.fastjson.JSONObject;
import com.hy.common.utils.AccessTokenUtils;
import com.hy.constant.Constant;

/**
 * 通用weichat URL description： author：JIAZHIPENG time：2016-9-8 下午2:51:50
 * 修改时间：2016-9-8 下午2:51:50 修改备注：
 */
public class WechatUrl {
	
	private static Logger logger = LoggerFactory.getLogger(WechatUrl.class);

	/**
	 * 
	 * @param appid
	 *            APP ＩＤ
	 * @param reirectUrl
	 *            重定向URL
	 * @param stateParam
	 *            　附带参数，可以传输订单号，最后微信会自动将这个参数拼接在重定向ＵＲＬ后面，
	 *            类似state=2015555,服务器就可以request.getParameter("state")获取订单号
	 * @return
	 * @throws UnsupportedEncodingException
	 */
	public static String buildAuthUrl(String appid, String reirectUrl,
			String stateParam) throws UnsupportedEncodingException {
		String url = "https://open.weixin.qq.com/connect/oauth2/authorize?appid=APPID&redirect_uri=REDIRECT_URI&response_type=code&scope=SCOPE&state=STATE#wechat_redirect";
		url = url
				.replace("APPID", appid)
				.replace("REDIRECT_URI", URLEncoder.encode(reirectUrl, "UTF-8"))
				.replace("SCOPE", "snsapi_userinfo")
				.replace("STATE", stateParam);
		return url;
	}

	/**
	 * 刷新access_token，有效期2分钟，刷新后至少一周。
	 * 
	 * @return {"access_token""expires_in""refresh_token""openid""scope"}
	 * @throws IOException
	 */
	public static String refreshAccessToken(String appid, String refreshToken,
			String method) throws IOException {
		String url = "https://api.weixin.qq.com/sns/oauth2/refresh_token?appid=APPID&grant_type=refresh_token&refresh_token=REFRESH_TOKEN";
		url = url.replace("APPID", appid)
				.replace("REFRESH_TOKEN", refreshToken);
		return getRemoteJson(url, method);
	}

	/**
	 * 验证用户授权token是否有效
	 * 
	 * @param token
	 * @param openid
	 * @param method
	 * @return { "errcode":0,"errmsg":"ok"}
	 * @throws IOException
	 */
	public static String validateToken(String token, String openid,
			String method) throws IOException {
		String url = "https://api.weixin.qq.com/sns/auth?access_token=ACCESS_TOKEN&openid=OPENID";
		url = url.replace("ACCESS_TOKEN", token).replace("OPENID", openid);
		return getRemoteJson(url, method);
	}

	/**
	 * 获取微信用户的个人信息
	 * 
	 * @param token
	 * @param openid
	 * @param method
	 * @return 
	 *         "openid","nickname""sex""province""city""country""headimgurl""privilege"
	 *         "unionid"
	 * @throws IOException
	 */
	public static String getUserinfo(String token, String openid, String method)
			throws IOException {
		String url = "https://api.weixin.qq.com/sns/userinfo?access_token=ACCESS_TOKEN&openid=OPENID&lang=zh_CN";
		url = url.replace("ACCESS_TOKEN", token).replace("OPENID", openid);
		return getRemoteJson(url, method);
	}

	/**
	 * 获取用户授权d的access_token
	 * 
	 * @return 
	 *         {"access_token""expires_in""refresh_token""openid""scope""unionid"
	 *         }
	 * @throws IOException
	 */
	public static String fetchAuthReturnMsg(String appid, String appSecret,
			String code, String method) throws IOException {
		String url = "https://api.weixin.qq.com/sns/oauth2/access_token?appid=APPID&secret=SECRET&code=CODE&grant_type=authorization_code";
		url = url.replace("APPID", appid).replace("SECRET", appSecret)
				.replace("CODE", code);
		// 刷新token
		return getRemoteJson(url, method);
	}

	public static InputStream getStringStream(String sInputString) {
		ByteArrayInputStream tInputStringStream = null;
		if (sInputString != null && !sInputString.trim().equals("")) {
			tInputStringStream = new ByteArrayInputStream(
					sInputString.getBytes());
		}
		return tInputStringStream;
	}

	/**
	 * 获取微信服务器返回的json字符串
	 * 
	 * @param targetUrl
	 * @param method
	 * @return
	 * @throws IOException
	 */
	public static String getRemoteJson(String targetUrl, String method)
			throws IOException {
		URL uri = new URL(targetUrl);
		String userAgent = "Mozilla/5.0 (Windows NT 6.1) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/29.0.1547.66 Safari/537.36";// 模拟浏览器
		HttpURLConnection connection = (HttpURLConnection) uri.openConnection();
		connection.setRequestMethod(method);
		connection.setReadTimeout(30000);
		connection.setConnectTimeout(30000);
		connection.setRequestProperty("User-agent", userAgent);
		connection.connect();
		InputStream is = connection.getInputStream();
		BufferedInputStream bufferedInputStream = new BufferedInputStream(is);
		StringBuffer buffer = new StringBuffer();
		byte[] bts = new byte[1024];
		int len = bufferedInputStream.read(bts);
		while (len > 0) {
			buffer.append(new String(bts, 0, len, "UTF-8"));
			len = bufferedInputStream.read(bts);
		}
		bufferedInputStream.close();
		return buffer.toString();
	}

	/**
	 * 获取一定长度的随机字符串
	 * 
	 * @param length
	 *            指定字符串长度
	 * @return 一定长度的字符串
	 */
	public static String getRandomStringByLength(int length) {
		String base = "abcdefghijklmnopqrstuvwxyz0123456789";
		Random random = new Random();
		StringBuffer sb = new StringBuffer();
		for (int i = 0; i < length; i++) {
			int number = random.nextInt(base.length());
			sb.append(base.charAt(number));
		}
		return sb.toString();
	}

	/**
	 * 获取 JS API调用的临时票据ticket
	 * 
	 * @param token
	 *            普通接口调用的access token
	 * @return
	 * @throws IOException
	 */
	private static String getJSApiTicket(String token) throws IOException {
		String url = "https://api.weixin.qq.com/cgi-bin/ticket/getticket?access_token=ACCESS_TOKEN&type=jsapi";
		url = url.replace("ACCESS_TOKEN", token);
		String ticketJson = getRemoteJson(url, "GET");
		// System.out.println("JS API调用的临时票据ticket响应结果:"+ticketJson);
		JSONObject json = JSONObject.parseObject(ticketJson);
		String errmsg = (String) json.get("errmsg");
		String ticket = (String) json.get("ticket");
		return "ok".equals(errmsg) ? ticket : "ticket error";
	}

	private static String byteToHex(final byte[] hash) {
		Formatter formatter = new Formatter();
		for (byte b : hash) {
			formatter.format("%02x", b);
		}
		String result = formatter.toString();
		formatter.close();
		return result;
	}

	/**
	 * 微信JS SDK调用签名
	 * 
	 * @param jsapi_ticket
	 * @param url
	 * @return
	 */
	public static Map<String, String> sign(String jsapi_ticket, String url,
			String nonce_str, String timestamp) {
		Map<String, String> ret = new HashMap<String, String>();
		String string1;
		String signature = "";
		// 注意这里参数名必须全部小写，且必须有序
		string1 = "jsapi_ticket=" + jsapi_ticket + "&noncestr=" + nonce_str
				+ "&timestamp=" + timestamp + "&url=" + url;
		try {
			MessageDigest crypt = MessageDigest.getInstance("SHA-1");
			crypt.reset();
			crypt.update(string1.getBytes("UTF-8"));
			signature = byteToHex(crypt.digest());
		} catch (NoSuchAlgorithmException e) {
			e.printStackTrace();
		} catch (UnsupportedEncodingException e) {
			e.printStackTrace();
		}
		ret.put("url", url);
		ret.put("ticket", jsapi_ticket);
		ret.put("signStr", nonce_str);
		ret.put("timestamp", timestamp);
		ret.put("sign", signature);
		return ret;
	}

	/**
	 * 获取JS SDK调用页面的初始化参数
	 * 
	 * @param appId
	 * @param appSecret
	 * @param apiCalledUrl
	 * @param timestamp
	 * @param weiPaySignStr
	 * @return
	 */
	public static Map<String, String> FetchConfigParams(String apiCalledUrl,
			String weiPaySignStr, String timestamp) {
		Map<String, String> configParam = new HashMap<String, String>();
		try {
			// 通过普通接口调用凭证获取jsapi_ticket
			String ticket = getJSApiTicket(AccessTokenUtils.getAccessToken());
			// 根据ticket来获取JS SDK初始化所需要的参数
			configParam = sign(ticket, apiCalledUrl, weiPaySignStr, timestamp);
		} catch (IOException e) {
			e.printStackTrace();
		}
		return configParam;
	}

	/**
	 * 获取统一下单编号
	 * 
	 * @param xmlParam
	 * @return
	 * @throws IOException
	 */
	public static String getPrepayId(String xmlParam) throws IOException {
		URL orderUrl = new URL("https://api.mch.weixin.qq.com/pay/unifiedorder");
		logger.info("获取统一下单编号"+orderUrl);
		HttpURLConnection conn = (HttpURLConnection) orderUrl.openConnection();
		conn.setConnectTimeout(30000); // 设置连接主机超时（单位：毫秒)
		conn.setReadTimeout(30000); // 设置从主机读取数据超时（单位：毫秒)
		conn.setDoOutput(true); // post请求参数要放在http正文内，顾设置成true，默认是false
		conn.setDoInput(true); // 设置是否从httpUrlConnection读入，默认情况下是true
		conn.setUseCaches(false); // Post 请求不能使用缓存

		conn.setRequestProperty("Content-Type", "text/xml");
		conn.setRequestMethod("POST");// 设定请求的方法为"POST"，默认是GET
		conn.setRequestProperty("Content-Length", xmlParam.length() + "");
		String encode = "UTF-8";
		OutputStreamWriter out = new OutputStreamWriter(conn.getOutputStream(),
				encode);
		logger.info("获取统一下单内容"+out);
		out.write(xmlParam);
		out.flush();
		out.close();
		// 获取响应内容体
		InputStream is = conn.getInputStream();
		BufferedInputStream bufferedInputStream = new BufferedInputStream(is);
		StringBuffer buffer = new StringBuffer();
		byte[] bts = new byte[1024];
		int len = bufferedInputStream.read(bts);
		while (len > 0) {
			buffer.append(new String(bts, 0, len));
			len = bufferedInputStream.read(bts);
		}
		logger.info("获取统一下单内容"+buffer);
		bufferedInputStream.close();
		try {
			Map<String, Object> orderMap = XMLParser.getMapFromXML(buffer
					.toString().trim());
			String prepay_id = (String) orderMap.get("prepay_id");
			logger.info("获取统一下单prepayId"+prepay_id);
			return prepay_id;
		} catch (ParserConfigurationException e) {
			e.printStackTrace();
			return "";
		} catch (SAXException e) {
			e.printStackTrace();
			return "";
		}
	}
	
	/**
	 * 查询订单
	 * 
	 * @param xmlParam
	 * @return
	 * @throws IOException
	 */
	public static Map<String, Object> getOrder(String xmlParam) throws IOException {
		URL orderUrl = new URL("https://api.mch.weixin.qq.com/pay/orderquery");
		logger.info(" 查询订单"+orderUrl);
		HttpURLConnection conn = (HttpURLConnection) orderUrl.openConnection();
		conn.setConnectTimeout(30000); // 设置连接主机超时（单位：毫秒)
		conn.setReadTimeout(30000); // 设置从主机读取数据超时（单位：毫秒)
		conn.setDoOutput(true); // post请求参数要放在http正文内，顾设置成true，默认是false
		conn.setDoInput(true); // 设置是否从httpUrlConnection读入，默认情况下是true
		conn.setUseCaches(false); // Post 请求不能使用缓存

		conn.setRequestProperty("Content-Type", "text/xml");
		conn.setRequestMethod("POST");// 设定请求的方法为"POST"，默认是GET
		conn.setRequestProperty("Content-Length", xmlParam.length() + "");
		String encode = "UTF-8";
		OutputStreamWriter out = new OutputStreamWriter(conn.getOutputStream(),
				encode);
		logger.info(" 查询订单内容"+out);
		out.write(xmlParam);
		out.flush();
		out.close();
		// 获取响应内容体
		InputStream is = conn.getInputStream();
		BufferedInputStream bufferedInputStream = new BufferedInputStream(is);
		StringBuffer buffer = new StringBuffer();
		byte[] bts = new byte[1024];
		int len = bufferedInputStream.read(bts);
		while (len > 0) {
			buffer.append(new String(bts, 0, len));
			len = bufferedInputStream.read(bts);
		}
		logger.info(" 查询订单内容"+buffer);
		bufferedInputStream.close();
		try {
			Map<String, Object> orderMap = XMLParser.getMapFromXML(buffer
					.toString().trim());
			return orderMap;
		} catch (ParserConfigurationException e) {
			logger.info(" 查询订单错误"+buffer);
			return null;
		} catch (SAXException e) {
			logger.info(" 查询订单错误"+buffer);
			return null;
		}
	}
}
