package com.hy.weixin.pay.util;

import java.io.BufferedReader;
import java.io.File;
import java.io.FileInputStream;
import java.io.IOException;
import java.io.InputStreamReader;
import java.security.KeyStore;
import java.util.Map;
import java.util.SortedMap;
import java.util.TreeMap;
import javax.net.ssl.SSLContext;
import org.apache.http.HttpEntity;
import org.apache.http.client.methods.CloseableHttpResponse;
import org.apache.http.client.methods.HttpPost;
import org.apache.http.conn.ssl.SSLConnectionSocketFactory;
import org.apache.http.conn.ssl.SSLContexts;
import org.apache.http.entity.StringEntity;
import org.apache.http.impl.client.CloseableHttpClient;
import org.apache.http.impl.client.HttpClients;
import org.apache.http.util.EntityUtils;
import org.apache.log4j.Logger;
import org.springframework.web.bind.annotation.RequestMapping;
import com.alibaba.fastjson.JSONObject;
import com.google.common.collect.Maps;
import com.hy.common.utils.JsonUtil;
import com.hy.constant.Constant;
import com.hy.weixin.http.HttpClientConnectionManager;
import com.hy.weixin.pay.entity.WeiXinUnifiedorder;
import com.hy.weixin.utils.CommonUtil;
import com.hy.weixin.utils.GetWxOrderno;
import com.hy.weixin.utils.SignUtil;
import com.hy.weixin.utils.TenpayUtil;


/**
 * 微信支付工具类
 * @author 海易出行
 *
 */
public class WeiXinPayUtil {
	
	
	protected static Logger logger = Logger.getLogger(WeiXinPayUtil.class);  

	
	/**
	 * 微信支付通知
	 * @param weiXinUnifiedorder
	 * @return
	 */
	@SuppressWarnings("static-access")
	public static Map<String,String> unifiedorder(WeiXinUnifiedorder weiXinUnifiedorder){
		//公众账号ID
		String appid = Constant.APPID;
		//商户号
		String mch_id = Constant.MCHID;
		//TODO 随机字符串	
		String nonce_str = getNonceStr();
		//TODO body 商品描述
		String body = weiXinUnifiedorder.getBody();
		//TODO 商户订单号 out_trade_no
		//String out_trade_no = RandomStringUtils.randomNumeric(9);
		String out_trade_no = weiXinUnifiedorder.getOutTradeNo();

		//TODO total_fee 总金额
		String total_fee = weiXinUnifiedorder.getTotalFee();
		//TODO spbill_create_ip 终端ip地址
		//String spbill_create_ip = request.getRemoteAddr();
		String spbill_create_ip = weiXinUnifiedorder.getSpbillCreateIp();
		//TODO 通知地址	notify_url
		String notify_url = Constant.load.getProperty("weixin.notify_url"); 
		//交易类型	trade_type	是
		String trade_type = "JSAPI";
		String openid = weiXinUnifiedorder.getOpenId();//用户标识	openid  jsapi必传
		//TODO sign 签名
		
		SortedMap<String, String> packageParams = new TreeMap<String, String>();
		packageParams.put("appid", appid);  
		packageParams.put("mch_id", mch_id);  
		packageParams.put("nonce_str", nonce_str);  
		packageParams.put("body", body);  
		packageParams.put("out_trade_no", out_trade_no);  
		packageParams.put("total_fee", total_fee);  
		packageParams.put("spbill_create_ip", spbill_create_ip);  
		packageParams.put("notify_url", notify_url);  
		packageParams.put("trade_type", trade_type);  
		packageParams.put("openid", openid);  
		String paySign = null;
		Map<String,Object>  map = null;
		
		SortedMap<String ,String> signMap = new TreeMap<String ,String >();
        try {
			paySign = SignUtil.getPayCustomSign(packageParams,Constant.WECHAT_KEY);
			packageParams.put("sign",paySign);
		    String xml = CommonUtil.ArrayToXml(packageParams); 
			String createOrderURL = "https://api.mch.weixin.qq.com/pay/unifiedorder";
			map = new GetWxOrderno().getMap(createOrderURL, xml);
			if(map == null){
				signMap.put("saveMessage","fail");
				return signMap;
			}
		} catch (Exception e) {
			e.printStackTrace();
		}
		//封装h5页面调用参数
		long timestamp = System.currentTimeMillis() / 1000;
        signMap.put("appId", appid);
        signMap.put("timeStamp", timestamp + "");
        signMap.put("package", "prepay_id=" + map.get("prepay_id"));
        signMap.put("signType", "MD5");
        signMap.put("nonceStr", map.get("nonce_str") + "");
        
        String finalsign = null;
		try {
			finalsign = SignUtil.getPayCustomSign(signMap,Constant.WECHAT_KEY);
		} catch (Exception e) {
			e.printStackTrace();
		}
		signMap.put("finalsign",finalsign );
		signMap.put("saveMessage","success");
		logger.info("----signMap----" + JsonUtil.extractJson(signMap));
        return signMap;
		
	}
	
	
	/**
	 * 微信支付退款
	 * @param request
	 * @return
	 * @throws Exception 
	 */
	@SuppressWarnings({ "unchecked", "deprecation" })
	@RequestMapping("/payRefund")
	public static Map<String,Object> payRefund(WeiXinUnifiedorder weiXinUnifiedorder) throws Exception{
		CloseableHttpResponse response = null;
		CloseableHttpClient httpclient = null;
		logger.info("---weiXinUnifiedorder---"+ JSONObject.toJSONString(weiXinUnifiedorder));
		String mch_id = weiXinUnifiedorder.getMchId();//不一样
		StringBuffer textBuffer = new StringBuffer("");
		try{
//			String basePath = request.getSession().getServletContext().getRealPath("WEB-INF/classes/certificate");
			String basePath = weiXinUnifiedorder.getBasePath();
			KeyStore keyStore = KeyStore.getInstance("PKCS12");
			FileInputStream instream = new FileInputStream(new File(basePath + weiXinUnifiedorder.getCertificate()));//不一样
			keyStore.load(instream, mch_id.toCharArray());
			SSLContext sslcontext = SSLContexts.custom().loadKeyMaterial(keyStore, mch_id.toCharArray()).build();
			SSLConnectionSocketFactory sslsf = new SSLConnectionSocketFactory(sslcontext, new String[] { "TLSv1" }, null,SSLConnectionSocketFactory.BROWSER_COMPATIBLE_HOSTNAME_VERIFIER);
			httpclient = HttpClients.custom().setSSLSocketFactory(sslsf).build();
			HttpPost httpost = HttpClientConnectionManager.getPostMethod("https://api.mch.weixin.qq.com/secapi/pay/refund");
			
			
			Map<String,String> map = Maps.newHashMap();
			map.put("appid", weiXinUnifiedorder.getAppid());		// 不一样公众账号ID 	appid 	是 	String(32) 	wx8888888888888888 	微信分配的公众账号ID（企业号corpid即为此appId）
			map.put("mch_id", mch_id);//商户号 	mch_id 	是 	String(32) 	1900000109 	微信支付分配的商户号
			map.put("nonce_str", getNonceStr());	//随机字符串 	nonce_str 	是 	String(32) 	5K8264ILTKCH16CQ2502SI8ZNMTM67VS 	随机字符串，不长于32位。推荐随机数生成算法
			map.put("transaction_id", weiXinUnifiedorder.getTransactionId());	//商户订单号 	out_trade_no 	String(32) 	1217752501201407033233368018 	商户侧传给微信的订单号
			map.put("out_refund_no", weiXinUnifiedorder.getTransactionId());	//商户退款单号 	out_refund_no 	是 	String(32) 	1217752501201407033233368018 	商户系统内部的退款单号，商户系统内部唯一，同一退款单号多次请求只退一笔
			map.put("total_fee", weiXinUnifiedorder.getTotalFee());				//总金额 	total_fee 	是 	Int 	100 	订单总金额，单位为分，只能为整数，详见支付金额
			map.put("refund_fee",weiXinUnifiedorder.getTotalFee());				//退款金额 	refund_fee 	是 	Int 	100 	退款总金额，订单总金额，单位为分，只能为整数，详见支付金额
			map.put("op_user_id", mch_id);//操作员 	op_user_id 	是 	String(32) 	1900000109 	操作员帐号, 默认为商户号
			String sign =  SignUtil.getPayCustomSign(map,weiXinUnifiedorder.getSecretKey());
			map.put("sign", sign);					//签名 	sign 	是 	String(32) 	C380BEC2BFD727A4B6845133519F3AD6 	签名，详见签名生成算法

		    String xml = CommonUtil.ArrayToXml(map); 
			httpost.setEntity(new StringEntity(xml, "UTF-8"));
			System.out.println("executing request" + httpost.getRequestLine());
			response = httpclient.execute(httpost);
			HttpEntity entity = response.getEntity();
			logger.info("----xml-----" + xml);
			logger.info(response.getStatusLine());
			
			
			if (entity != null) {
				logger.info("Response content length: " + entity.getContentLength());
				BufferedReader bufferedReader = new BufferedReader(new InputStreamReader(entity.getContent()));
				String text = "";
				while ((text = bufferedReader.readLine()) != null) {
					textBuffer.append(text);
				}
				logger.info("返回的信息...." + textBuffer.toString());

			}
			EntityUtils.consume(entity);
		}catch(Exception e){
			logger.error("微信支付退款异常.........",e);
		}finally {
			try {
				response.close();
				httpclient.close();
			} catch (IOException e) {
				e.printStackTrace();
			}
		}
		return GetWxOrderno.doXMLParse(textBuffer.toString());
	} 
	
	
	
	/**
	 * 获取随机数
	 * @return
	 */
	private static String getNonceStr() {
		String currTime = TenpayUtil.getCurrTime();
		//8位日期
		String strTime = currTime.substring(8, currTime.length());
		//四位随机数
		String strRandom = TenpayUtil.buildRandom(4) + "";
		//10位序列号,可以自行调整。
		String strReq = strTime + strRandom;
		//随机数 
		String nonce_str = strReq;
		return nonce_str;
	}
	
	
	/**
	 * 验证签名
	 * @param strmap
	 * @return
	 * @throws Exception
	 */
	public static boolean verifySign(Map<String,String> strmap) throws Exception{
		String sign = strmap.get("sign").toString();//微信发过来的签名
		strmap.remove("sign");
		String finalsign = SignUtil.getPayCustomSign(strmap,Constant.load.getProperty("weixin.pay.key"));
		return sign.equalsIgnoreCase(finalsign);
		
	}

}
