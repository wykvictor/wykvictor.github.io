---
layout: post
title:  "C Redirect Stdout to File"
date:   2018-10-08 16:00:00
tags: [recirect, stdout, file, freopen]
categories: Tech
---

> 为了在C代码里实现类似Shell脚本的功能: ./exe 2>&1 | tee log.txt

### 1. Use freopen
{% highlight C++ %}
// output console and file simutainiously
class RedirectStdIO {
public:
    RedirectStdIO(string str, bool needStd = true): path(str), need_std(needStd) {
        std_out = dup(fileno(stdout));  // 保存下来，方便后续找回std_out
        std_err = dup(fileno(stderr));
        if(!direct2file(path))
            std::cerr << "Can not redirect output to file " << path << std::endl;
    }
    bool direct2file(string str){
        if(str.empty()) return false;
        fp = freopen(str.c_str(), "w", stdout);
        dup2(fileno(stdout), fileno(stderr));  // 重定向
        return (fp!=nullptr);
    }
    void back2stdout() {
        fflush(fp);
        dup2(std_out, fileno(stdout));  // cout重回screen
        dup2(std_err, fileno(stderr));
        close(std_out);
        close(std_err);
        // cout lines in file path to stdout
        string line;
        ifstream myfile(path);
        if (myfile.is_open()) {
            while ( getline (myfile,line) ) {
                cout << line << '\n';
            }
            myfile.close();
        }
        cout.flush();
        need_std = false;
    }
    ~RedirectStdIO(){
        if(need_std) back2stdout();
        if(fp) fflush(fp);
        if(fp) fclose(fp);
    }
private:
    FILE *fp = nullptr;
    int std_out, std_err;
    bool need_std;
    string path;
};
{% endhighlight %}

### 2. Use c11 streambuf
[referenc](http://forums.codeguru.com/showthread.php?505676-streambuf/page2)

[reference2](http://kaiyuan.me/2017/06/22/custom-streambuf/)
{% highlight C++ %}
#include <iostream>
#include <streambuf>
#include <fstream>

//------------------------------------------------------------------------------

template <class charT, class traits = std::char_traits<charT> >
class basic_teebuf : public std::basic_streambuf<charT, traits>
{
public:
    typedef charT char_type;
    typedef typename traits::int_type int_type;
    typedef typename traits::pos_type pos_type;
    typedef typename traits::off_type off_type;
    typedef traits traits_type;
    typedef std::basic_streambuf<charT, traits> streambuf_type;

private:
    streambuf_type *m_sbuf1;
    streambuf_type *m_sbuf2;
    char_type *m_buffer;
    enum {BUFFER_SIZE = 4096 / sizeof(char_type)};

public:
    basic_teebuf(streambuf_type *sbuf1, streambuf_type *sbuf2)
        : m_sbuf1(sbuf1), m_sbuf2(sbuf2), m_buffer(new char_type[BUFFER_SIZE])
    {
        setp(m_buffer, m_buffer + BUFFER_SIZE);
    }//constructor
    ~basic_teebuf()
    {
        this->pubsync();
        delete[] m_buffer;
    }//destructor
protected:
    virtual int_type overflow(int_type c = traits_type::eof())
    {
        // empty our buffer into m_sbuf1 and m_sbuf2
        std::streamsize n = static_cast<std::streamsize>(this->pptr() - this->pbase());
        std::streamsize size1 = m_sbuf1->sputn(this->pbase(), n);
        std::streamsize size2 = m_sbuf2->sputn(this->pbase(), n);
        if (size1 != n || size2 != n)
            return traits_type::eof();
        // reset our buffer
        setp(m_buffer, m_buffer + BUFFER_SIZE);
        // write the passed character if necessary
        if (!traits_type::eq_int_type(c, traits_type::eof()))
        {
            traits_type::assign(*this->pptr(), traits_type::to_char_type(c));
            this->pbump(1);
        }//if
        return traits_type::not_eof(c);
    }//overflow

    virtual int sync()
    {
        // flush our buffer into m_sbuf1 and m_sbuf2
        int_type c = this->overflow(traits_type::eof());
        // checking return for eof.
        if (traits_type::eq_int_type(c, traits_type::eof()))
            return -1;
        // flush m_sbuf1 and m_sbuf2
        if (m_sbuf1->pubsync() == -1 || m_sbuf2->pubsync() == -1)
            return -1;
        return 0;
    }//sync
};//basic_teebuf
typedef basic_teebuf<char>    teebuf;
typedef basic_teebuf<wchar_t> wteebuf;
//------------------------------------------------------------------------------

template <class charT, class traits = std::char_traits<charT> >
struct scoped_basic_streambuf_assignment
{
    typedef std::basic_ios<charT, traits> stream_type;
    typedef std::basic_streambuf<charT, traits> streambuf_type;
    stream_type &m_s;
    streambuf_type *m_orig_sb;
    scoped_basic_streambuf_assignment(stream_type &s, streambuf_type *new_sb) 
        : m_s(s)
    {
        m_orig_sb = m_s.rdbuf(new_sb);
    }//constructor
    ~scoped_basic_streambuf_assignment()
    {
        m_s.rdbuf(m_orig_sb);
    }//destructor
};//scoped_streambuf_assignment
typedef scoped_basic_streambuf_assignment<char>    scoped_streambuf_assignment;
typedef scoped_basic_streambuf_assignment<wchar_t> scoped_wstreambuf_assignment;
//------------------------------------------------------------------------------
int main()
{
    std::ofstream file("data.txt");
    std::ofstream file2("data2.txt");
    // tbuf will write to both file and cout
    teebuf tbuf(file.rdbuf(), std::cout.rdbuf());
    // tbuf2 will write to both tbuf and file2
    teebuf tbuf2(file2.rdbuf(), &tbuf);
    // replace cout's streambuf with tbuf2
    scoped_streambuf_assignment ssa(std::cout, &tbuf2);
    std::cout << "Hello World" << std::endl;
    return 0;
}//main
{% endhighlight %}
