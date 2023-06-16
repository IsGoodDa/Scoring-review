以下是Java代码的一个示例；The following is an example of Java code：

import java.sql.Connection;

import java.sql.DriverManager;

import java.sql.PreparedStatement;

import java.sql.ResultSet;

import java.sql.SQLException;


public class ScoreAudit {
    private int id;
    private int score;
    private String auditor;
    private String date;

    // 构造函数
    public ScoreAudit(int id, int score, String auditor, String date) {
        this.id = id;
        this.score = score;
        this.auditor = auditor;
        this.date = date;
    }

    // 审核评分
    public static void auditScore(ScoreAudit audit) {
        Connection conn = null;
        PreparedStatement pstmt = null;

        try {
            // 连接数据库
            conn = DriverManager.getConnection("jdbc:mysql://localhost:3306/mydb", "root", "password");

            // 拼接 SQL 语句
            String sql = "UPDATE scores SET auditor=?, date=? WHERE id=?";
            pstmt = conn.prepareStatement(sql);

            // 设置参数
            pstmt.setString(1, audit.getAuditor());
            pstmt.setString(2, audit.getDate());
            pstmt.setInt(3, audit.getId());

            // 执行 SQL 语句
            pstmt.executeUpdate();

            System.out.println("审核评分成功！");
        } catch (SQLException e) {
            e.printStackTrace();
        } finally {
            // 关闭连接
            try {
                if (pstmt != null) pstmt.close();
                if (conn != null) conn.close();
            } catch (SQLException e) {
                e.printStackTrace();
            }
        }
    }

    // 查看所有评分审核记录
    public static ScoreAudit[] getAllScoreAudits() {
        Connection conn = null;
        PreparedStatement pstmt = null;
        ResultSet rs = null;
        ScoreAudit[] audits = null;

        try {
            // 连接数据库
            conn = DriverManager.getConnection("jdbc:mysql://localhost:3306/mydb", "root", "password");

            // 拼接 SQL 语句
            String sql = "SELECT * FROM scores WHERE auditor IS NOT NULL";
            pstmt = conn.prepareStatement(sql);
            rs = pstmt.executeQuery();

            // 获取记录个数
            rs.last();
            int numRows = rs.getRow();
            audits = new ScoreAudit[numRows];
            rs.beforeFirst();

            // 解析查询结果
            int i = 0;
            while (rs.next()) {
                int id = rs.getInt("id");
                int score = rs.getInt("score");
                String auditor = rs.getString("auditor");
                String date = rs.getString("date");

                ScoreAudit audit = new ScoreAudit(id, score, auditor, date);
                audits[i] = audit;
                i++;
            }
        } catch (SQLException e) {
            e.printStackTrace();
        } finally {
            // 关闭连接
            try {
                if (rs != null) rs.close();
                if (pstmt != null) pstmt.close();
                if (conn != null) conn.close();
            } catch (SQLException e) {
                e.printStackTrace();
            }
        }

        return audits;
    }

    // getters and setters
    public int getId() {
        return id;
    }

    public void setId(int id) {
        this.id = id;
    }

    public int getScore() {
        return score;
    }

    public void setScore(int score) {
        this.score = score;
    }

    public String getAuditor() {
        return auditor;
    }

    public void setAuditor(String auditor) {
        this.auditor = auditor;
    }

    public String getDate() {
        return date;
    }

    public void setDate(String date) {
        this.date = date;
    }
}
