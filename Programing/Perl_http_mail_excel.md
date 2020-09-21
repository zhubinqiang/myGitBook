# Perl 常用工具代码

[TOC]

## http
```perl
#!/usr/bin/env perl
use strict;
use warnings;
use 5.010;
use HTTP::Request;
use HTTP::Cookies;
use LWP::UserAgent;
my $url = 'http://shilv018.example.com/bugzilla_iMediaPro/';

# cookie
my $cookie_jar  = HTTP::Cookies->new(
        file        => "./acookies.lwp",
        autosave    => 1,
        );
# login
my $ua      = LWP::UserAgent->new;
my $cookies = $ua->cookie_jar($cookie_jar);
$ua->agent('Mozilla/9 [en] (Centos; Linux)');
my $res     = $ua->post( $url,
    [
    Bugzilla_login => 'username',
    Bugzilla_password    => 'password',
    origURL   =>'http://shilv018.example.com/bugzilla_iMediaPro/',
    ],
);
$res =$ua->get('http://shilv018.example.com/bugzilla_iMediaPro/buglist.cgi?bug_status=__open__&list_id=9455&product=iMediaPro&query_format=specific&ctype=csv&human=1');

my $out = "save.csv";
open(FILE, ">$out") or die "can't open $out $!";
print FILE $res->content();
close FILE;
```

## 写入excel
```csv
ID,TU,MBBRC,PSNR
1,1,on,41.14999
2,2,on,41.1331
3,3,on,41.09997
4,4,on,41.12508
5,5,on,41.12508
6,6,on,41.09753
7,7,on,40.68288
8,1,off,41.33498
9,2,off,41.29093
10,3,off,41.2962
11,4,off,41.29707
12,5,off,41.29707
13,6,off,41.26188
14,7,off,40.87638
```

| ID | TU | MBBRC | PSNR     |
| :- | :- | :-    | :-       |
| 1  | 1  | on    | 41.14999 |
| 2  | 2  | on    | 41.1331  |
| 3  | 3  | on    | 41.09997 |
| 4  | 4  | on    | 41.12508 |
| 5  | 5  | on    | 41.12508 |
| 6  | 6  | on    | 41.09753 |
| 7  | 7  | on    | 40.68288 |
| 8  | 1  | off   | 41.33498 |
| 9  | 2  | off   | 41.29093 |
| 10 | 3  | off   | 41.2962  |
| 11 | 4  | off   | 41.29707 |
| 12 | 5  | off   | 41.29707 |
| 13 | 6  | off   | 41.26188 |
| 14 | 7  | off   | 40.87638 |


```perl
#!/usr/bin/env perl

use strict;
use Excel::Writer::XLSX;
use File::Find;
use File::Copy;
use File::stat;
use Cwd;
use Term::ANSIColor;

#unlink("abc");

my $excleName = "psnr.xlsx";

my $workbook = Excel::Writer::XLSX->new($excleName);
my $worksheet = $workbook->add_worksheet();
my $worksheet2 = $workbook->add_worksheet();

my $case_number = 0;

my $format1 = $workbook->add_format();
$format1->set_bold();

my $format2 = $workbook->add_format();
$format2->set_bg_color('red');

my $format3 = $workbook->add_format();
$format3->set_bg_color('yellow');

my $format4 = $workbook->add_format();
$format4->set_bg_color('blue');



my $csvName = "1.csv";
open(CSV, $csvName) or die "open $csvName : $!";

my $lineNo = 0;
while(<CSV>) {
    my @sp = split ',', $_;
    #print @sp[0];
    chomp($sp[16]);
    if($lineNo eq 0) {
        $worksheet->write(0, 0, $sp[0], $format1);
        $worksheet->write(0, 1, $sp[1], $format1);
        $worksheet->write(0, 2, $sp[2], $format1);
        $worksheet->write(0, 3, $sp[3], $format1);
    }else {
        $worksheet->write($lineNo, 0, "$sp[0]");
        $worksheet->write($lineNo, 1, "$sp[1]");
        $worksheet->write($lineNo, 2, "$sp[2]");
        $worksheet->write($lineNo, 3, "$sp[3]");
    }
    $lineNo += 1;
}

close(CSV);


my $chart = $workbook->add_chart(
    type => 'scatter',
    subtype => 'smooth_with_markers',
    embedded => 1);

my $chart_series_name1 = '=Sheet1!$C$2';
my $chart_series_category1 = '=Sheet1!$B$2:$B$8';
my $chart_series_value1= '=Sheet1!$D$2:$D$8';

$chart->add_series(
    name => $chart_series_name1,
    categories => $chart_series_category1,
    values => $chart_series_value1,
);


my $chart_series_name2 = '=Sheet1!$C$9';
my $chart_series_category2 = '=Sheet1!$B$9:$B$15';
my $chart_series_value2= '=Sheet1!$D$9:$D$15';

$chart->add_series(
    name => $chart_series_name2,
    categories => $chart_series_category2,
    values => $chart_series_value2,
);

$chart->set_title(name=>"720P", name_font=>{size=>11});
$chart->set_x_axis(name => 'TU');
$chart->set_y_axis(name => 'PSNR');
my $chart_series_name2 = '=Sheet1!$C$5';

$worksheet->insert_chart('E1', $chart, 0, 0) or warn $!;
```


## 发送邮件
```perl
#!/usr/bin/perl
use strict;
use warnings;
use MIME::Lite;

my $mailFrom='qabuild@example.com';
my $mailTo='user@example.com';
my $mail_file = "mail.txt";
printf $mailTo."\n";
open MF, ">$mail_file" || die "Can't open mail file \'$mail_file\': $!\n";

my $content = <<'END';
1
2
3
END

print $content;
print MF $content;
close MF;

my $subject = "Subject";
my $msg     = MIME::Lite->new(
                From     => $mailFrom,
                To       => $mailTo,
                Bcc      => $mailTo,
                Subject  => $subject,
                Type     => 'text/html',
                Encoding => 'base64',
                Path     => $mail_file,
                );
$msg->attach(
                Type    => 'text',
                Path    => "$mail_file",
                );
$msg->send( 'smtp', 'mail.example.com' );
```
